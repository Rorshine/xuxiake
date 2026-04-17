

<template>
  <div class="map-view" :class="{ 'detail-mode': isDetailMode }">
    <!-- 地图容器 -->
    <div ref="mapContainer" class="map-container">
      <img src="/images/south.png" alt="南海" class="south-china-sea-image" />
    </div>
    <!-- 右侧卡片（仅详情模式显示） -->
    <div v-if="isDetailMode" class="story-cards">
      <div class="card">
        <div class="card-header">
          <h3 class="title">地点信息</h3>
          <div class="hint">点击时间轴下方的卡片查看</div>
        </div>
        <p class="inner-content" v-html="placeInfoDisplay"></p>
      </div>
      <div class="card">
        <div class="card-header">
          <h3 class="title">重要事件</h3>
          <div class="hint">支持滚动查看完整内容</div>
        </div>
        <p class="inner-content" v-html="placeImpoDisplay"></p>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "MapView",
  props: {
    activeChapter: { type: Number, default: null },
    selectedChapters: { type: Array, default: () => [] },
    isDetailMode: { type: Boolean, default: false },
    locations: { type: Array, required: true },
    timeData: { type: Array, required: true },
    chapters: { type: Array, required: true },
  },
  data() {
    return {
      svg: null,
      projection: null,
      initFlag: false,
      timelineY: 0,
      timelineX: 0,
      len: 4,
      placeInfoDisplay: "这里会显示地点信息",
      placeImpoDisplay: "这里会显示重要事件信息",
      theme: {
        ink: "#1f2424",
        inkWeak: "rgba(31, 36, 36, 0.55)",
        paperStroke: "rgba(31, 36, 36, 0.12)",
        landFill: "#dcd1b2",
        landHover: "#cbbf9b",
        routeBase: "rgba(31, 36, 36, 0.35)",
        routeActive: "#b33a2b",
        pointBase: "rgba(31, 36, 36, 0.55)",
        pointActive: "#b33a2b",
        timeAxis: "rgba(31, 36, 36, 0.22)",
        timeDot: "rgba(179, 58, 43, 0.55)",
        timeLink: "rgba(179, 58, 43, 0.45)",
      },
      resizeObserver: null,
      /**
       * 地图放大系数（在 Mercator 的 scale 上乘以该值）。
       * 缩略与详情共用；想更大可调到 1.15～1.25，若裁边则略减小。
       */
      mapProjectionScaleK: 1.3,
    };
  },
  watch: {
    activeChapter: {
      handler() {
        if (this.svg && this.projection) this.updateByChapter();
      },
      immediate: true,
    },
    isDetailMode: {
      handler() {
        this.$nextTick(() => this.resizeMap());
      },
    },
    locations: {
      handler() {
        if (this.svg && this.projection) this.updateByChapter();
      },
      deep: true,
    },
    timeData: {
      handler() {
        if (this.svg && this.projection) this.updateByChapter();
      },
      deep: true,
    },
    chapters: {
      handler() {
        if (this.svg && this.projection) this.updateByChapter();
      },
      deep: true,
    },
    selectedChapters: {
      handler() {
        if (this.svg && this.projection) this.updateByChapter();
      },
      deep: true,
    },
  },
  mounted() {
    this.initMap();
    this._resizeDebounce = null;
    this.resizeObserver = new ResizeObserver(() => {
      if (this._resizeDebounce) clearTimeout(this._resizeDebounce);
      this._resizeDebounce = setTimeout(() => {
        this._resizeDebounce = null;
        this.resizeMap();
      }, 80);
    });
    if (this.$refs.mapContainer) {
      this.resizeObserver.observe(this.$refs.mapContainer);
    }
  },
  beforeUnmount() {
    if (this._resizeDebounce) clearTimeout(this._resizeDebounce);
    if (this.resizeObserver) this.resizeObserver.disconnect();
  },
  methods: {
    seeded01(key) {
      const s = String(key);
      let h = 2166136261;
      for (let i = 0; i < s.length; i++) {
        h ^= s.charCodeAt(i);
        h = Math.imul(h, 16777619);
      }
      h ^= h << 13; h ^= h >>> 17; h ^= h << 5;
      return ((h >>> 0) % 10000) / 10000;
    },
    randSigned(key, amp) {
      return (this.seeded01(key) * 2 - 1) * amp;
    },

    /** 详情模式下为时间轴+矩形标签预留的底部高度（必须在 viewBox 高度内；越小地图主区越高） */
    timelineReservedBottom(height) {
      if (!this.isDetailMode) return 0;
      const h = Math.max(120, height);
      return Math.min(Math.max(Math.floor(h * 0.34), 148), Math.floor(h * 0.46));
    },

    // 重新调整投影并重绘整个地图（用于容器尺寸变化）
    resizeMap() {
      if (!this.$refs.mapContainer) return;
      const container = this.$refs.mapContainer;
      const width = container.clientWidth;
      const height = container.clientHeight;
      if (width === 0 || height === 0) return;

      const reserveBottom = this.timelineReservedBottom(height);
      const mapDrawH = Math.max(72, height - reserveBottom);
      this._mapDrawHeight = mapDrawH;
      this._reserveBottom = reserveBottom;

      const baseScale = mapDrawH * 0.92;
      //const yTranslate = this.isDetailMode ? mapDrawH / 1.4 : mapDrawH / 1.5;
      this.projection = d3.geoMercator()
        .center([107, 20])
        .scale(baseScale * this.mapProjectionScaleK)
        .translate([width / 1.85, mapDrawH / 1.08]);

      if (!this.svg || this.svg.empty()) {
        d3.select(container).selectAll("svg").remove();
        this.svg = d3.select(container)
          .append("svg")
          .attr("preserveAspectRatio", "xMidYMid meet")
          .attr("viewBox", `0 0 ${width} ${height}`);
        void this.drawMapPath().then(() => {
          this.drawAllCirclesAndLines();
          if (this.isDetailMode) {
            this.initTimelineBase();
          }
          this.updateByChapter();
          this.initFlag = true;
        });
        return;
      }

      this.svg
        .attr("viewBox", `0 0 ${width} ${height}`)
        .attr("preserveAspectRatio", "xMidYMid meet");
      this.svg.selectAll("*").remove();
      void this.drawMapPath().then(() => {
        this.drawAllCirclesAndLines();
        if (this.isDetailMode) {
          this.initTimelineBase();
        }
        this.updateByChapter();
        this.initFlag = true;
      });
      this.$nextTick(() => {
    if (this.$parent && this.$parent.$parent && this.$parent.$parent.fullpageInstance) {
      this.$parent.$parent.fullpageInstance.reBuild(); // 触发 fullpage 重新计算高度
    }
  });
    },

    async initMap() {
      const container = this.$refs.mapContainer;
      if (!container) return;
      await this.$nextTick();
      this.resizeMap();
    },

    drawMapPath() {
      return new Promise((resolve, reject) => {
        d3.json("/data/china.geo.json").then(mapData => {
          const path = d3.geoPath().projection(this.projection);
          this.svg.append("g")
            .selectAll("path")
            .data(mapData.features)
            .enter()
            .append("path")
            .attr("d", path)
            .style("fill", this.theme.landFill)
            .style("stroke", this.theme.paperStroke)
            .style("stroke-width", 0.6)
            .on("mouseover", (event, d) => {
              d3.select(event.currentTarget).style("fill", this.theme.landHover);
              this.createTooltip(event, d.properties.name);
            })
            .on("mouseout", (event) => {
              d3.select(event.currentTarget).style("fill", this.theme.landFill);
              this.removeTooltip();
            });
          resolve();
        }).catch(reject);
      });
    },

    drawAllCirclesAndLines() {
      this.svg.append("g").attr("class", "circles-group");
      this.svg.append("g").attr("class", "lines-group");
      this.updateCirclesAndLines();
    },

    updateCirclesAndLines() {
      if (!this.svg || this.svg.select(".circles-group").empty() || this.svg.select(".lines-group").empty()) {
        return;
      }
      const selectedIndexes = this.isDetailMode
        ? (this.activeChapter !== null ? [this.activeChapter] : [])
        : ((this.selectedChapters || []).length ? this.selectedChapters : (this.activeChapter !== null ? [this.activeChapter] : []));
      const selectedChapterNames = new Set(
        selectedIndexes
          .map(index => this.chapters[index])
          .filter(Boolean)
      );
      const selectedLocations = this.locations.filter(l => selectedChapterNames.has(l.chapter));

      const circlesGroup = this.svg.select(".circles-group");
      circlesGroup.selectAll("circle").remove();
      circlesGroup.selectAll("circle")
        .data(this.locations.filter(d => {
          const lon = Number(d.lon);
          const lat = Number(d.lat);
          return Number.isFinite(lon) && Number.isFinite(lat);
        }))
        .enter()
        .append("circle")
        .attr("cx", d => this.projection([Number(d.lon), Number(d.lat)])[0])
        .attr("cy", d => this.projection([Number(d.lon), Number(d.lat)])[1])
        .attr("r", 2.2)
        .attr("fill", d => {
          if (!this.initFlag) return this.theme.pointBase;
          return selectedLocations.includes(d) ? this.theme.pointActive : this.theme.pointBase;
        })
        .style("cursor", "pointer")
        .on("click", (event, d) => {
          if (d.chapter) this.$emit("chapter-change", d.chapter);
        });

      const linesGroup = this.svg.select(".lines-group");
      linesGroup.selectAll("line").remove();
      for (let i = 0; i < this.locations.length - 1; i++) {
        const start = this.locations[i];
        const end = this.locations[i + 1];
        const slon = Number(start.lon);
        const slat = Number(start.lat);
        const elon = Number(end.lon);
        const elat = Number(end.lat);
        if (![slon, slat, elon, elat].every(Number.isFinite)) continue;
        if (Math.abs(slon - elon) < this.len && Math.abs(slat - elat) < this.len) {
          const [sx, sy] = this.projection([slon, slat]);
          const [ex, ey] = this.projection([elon, elat]);
          const isActive = selectedLocations.includes(start) && selectedLocations.includes(end);
          linesGroup.append("line")
            .attr("x1", sx).attr("y1", sy)
            .attr("x2", ex).attr("y2", ey)
            .attr("stroke", isActive ? this.theme.routeActive : this.theme.routeBase)
            .attr("stroke-width", 1.6)
            .style("stroke-linecap", "round")
            .style("opacity", 0.78)
            .style("cursor", "pointer")
            .on("click", () => {
              const chapterName = start.chapter || end.chapter;
              if (chapterName) this.$emit("chapter-change", chapterName);
            });
        }
      }
    },

    initTimelineBase() {
      const mapSvg = this.svg;
      if (!mapSvg) return;
      const container = this.$refs.mapContainer;
      const mapHeight = container.clientHeight;
      const timelineWidth = container.clientWidth;
      const timelineHeight = 60;
      const mapH = this._mapDrawHeight != null ? this._mapDrawHeight : mapHeight;
      const axisLineY = Math.min(mapH + 18, mapHeight - 8);
      const timelineYPosition = axisLineY - timelineHeight / 2;
      this.timelineY = timelineYPosition;
      this.timelineX = timelineWidth;

      // 如果已有 timeline-group 则移除重建，避免重复
      mapSvg.select(".timeline-group").remove();
      const svg = mapSvg.append("g").attr("class", "timeline-group");
      svg.append("line")
        .attr("x1", 20)
        .attr("x2", timelineWidth - 20)
        .attr("y1", timelineYPosition + timelineHeight / 2)
        .attr("y2", timelineYPosition + timelineHeight / 2)
        .attr("stroke", this.theme.timeAxis)
        .attr("stroke-width", 2);
    },

    async updateByChapter() {
      if (!this.svg || !this.projection) return;
      const selectedChapterName = (this.activeChapter !== null) ? this.chapters[this.activeChapter] : null;
      const selectedTimeData = this.timeData.filter(item => item.chapter === selectedChapterName);
      
      // 更新地点和路线的颜色（无论缩略还是详情都需要）
      this.updateCirclesAndLines();

      // 清除时间轴组的所有内容（避免残留）
      const timelineGroup = this.svg.select(".timeline-group");
      if (!timelineGroup.empty()) {
        timelineGroup.selectAll("*").remove();
      }

      // 缩略模式：不绘制任何时间轴元素，直接返回
      if (!this.isDetailMode) {
        return;
      }

      // 以下是详情模式：绘制完整时间轴
      if (!selectedTimeData.length) return;

      // 重新添加时间轴基线（因为上面清空了）
      this.initTimelineBase();
      const newTimelineGroup = this.svg.select(".timeline-group");
      const timelineWidth = this.timelineX || this.svg.node().getBoundingClientRect().width;
      const timelineHeight = 60;
      const centerY = this.timelineY;

      const minDate = d3.min(selectedTimeData, d => new Date(d.time));
      const maxDate = d3.max(selectedTimeData, d => new Date(d.time));
      const timeSpan = maxDate - minDate;
      let adjustedMinDate, adjustedMaxDate;
      if (timeSpan <= 365 * 24 * 60 * 60 * 1000) {
        adjustedMinDate = d3.timeMonth.offset(minDate, -1);
        adjustedMaxDate = d3.timeMonth.offset(maxDate, 1);
      } else {
        adjustedMinDate = d3.timeYear.offset(minDate, -1);
        adjustedMaxDate = d3.timeYear.offset(maxDate, 1);
      }

      const xScale = d3.scaleTime()
        .domain([adjustedMinDate, adjustedMaxDate])
        .range([30, timelineWidth - 30]);

      const axisBottom = d3.axisBottom(xScale)
        .ticks(timeSpan <= 365*24*60*60*1000 ? d3.timeMonth.every(1) : d3.timeYear.every(1))
        .tickFormat(d3.timeFormat("%Y-%m"))
        .tickSizeInner(8)
        .tickSizeOuter(0);
      newTimelineGroup.append("g")
        .attr("class", "timeline-axis")
        .attr("transform", `translate(0, ${centerY + timelineHeight / 2})`)
        .call(axisBottom)
        .selectAll(".tick text")
        .style("font-size", "15px")
        .style("font-weight", "bold")
        .style("fill", "#000");
      newTimelineGroup.select(".timeline-axis path").style("display", "none");

      newTimelineGroup.selectAll(".timeline-point")
        .data(selectedTimeData)
        .enter()
        .append("circle")
        .attr("class", "timeline-point")
        .attr("cx", d => xScale(new Date(d.time)))
        .attr("cy", centerY + timelineHeight / 2)
        .attr("r", 5.5)
        .attr("fill", this.theme.timeDot)
        .style("opacity", 0.85);

      const lineGroup = newTimelineGroup.append("g").attr("class", "timeline-link");
      lineGroup.selectAll("path")
        .data(selectedTimeData)
        .enter()
        .append("path")
        .attr("d", d => {
          const start = [xScale(new Date(d.time)), centerY + timelineHeight / 2];
          const end = this.projection([Number(d.lon), Number(d.lat)]);
          if (!end || end.some(v => !Number.isFinite(v))) return "";
          const control = [(start[0] + end[0]) / 2, (start[1] + end[1]) / 2 - 50];
          return d3.line().curve(d3.curveCardinal).x(p => p[0]).y(p => p[1])([start, control, end]);
        })
        .attr("fill", "none")
        .attr("stroke", this.theme.timeLink)
        .attr("stroke-width", 2)
        .style("opacity", 0.5)
        .style("stroke-dasharray", "4,4");

      const mapHeight = this.$refs.mapContainer ? this.$refs.mapContainer.clientHeight : 0;
      const axisY = centerY + timelineHeight / 2;
      const roomBelow = Math.max(40, mapHeight - axisY - 8);
      this._labelScale = Math.min(1, Math.max(0.52, roomBelow / 218));

      this.drawRectLabels(selectedTimeData, xScale, centerY, timelineHeight, newTimelineGroup, lineGroup, timelineWidth, mapHeight);

      this.svg.select(".axis-label").remove();
      this.svg.append("text")
        .attr("class", "axis-label")
        .attr("x", timelineWidth * 0.95)
        .attr("y", centerY + timelineHeight / 2 - 10)
        .attr("text-anchor", "middle")
        .text("时间轴")
        .style("font-size", "16px")
        .style("font-weight", "bold");
    },


    drawRectLabels(selectedTimeData, xScale, centerY, timelineHeight, timelineGroup, lineGroup, timelineWidth, mapHeight) {
      const s = this._labelScale != null ? this._labelScale : 1;
      const rectWidth = 42;
      const rectHeight = Math.max(48, Math.round(92 * s));
      const laneStep = Math.max(62, Math.round(108 * s));
      const baseY = centerY + Math.round(72 * s);
      const laneCount = 2;
      const minGap = rectWidth + 18;
      const padX = 36;
      const xMin = padX + rectWidth / 2;
      const xMax = (timelineWidth - padX) - rectWidth / 2;

      const lastRight = new Array(laneCount).fill(-Infinity);
      const layout = selectedTimeData.map((d, i) => {
        const baseX = xScale(new Date(d.time));
        const key = `${d.name}-${d.time}-${i}`;

        const tryLane = (lane) => {
          const minCenter = lastRight[lane] === -Infinity ? xMin : (lastRight[lane] + minGap + rectWidth / 2);
          const x = Math.min(xMax, Math.max(baseX, minCenter));
          const shift = Math.abs(x - baseX);
          return { lane, x, shift };
        };
        const a = tryLane(0);
        const b = tryLane(1);
        const picked = a.shift <= b.shift ? a : b;
        const right = picked.x + rectWidth / 2;
        lastRight[picked.lane] = right;
        const jitter = this.randSigned(`${key}-jx`, 3);
        const xJ = Math.min(xMax, Math.max(xMin, picked.x + jitter));
        return { x: xJ, y: baseY + picked.lane * laneStep, lane: picked.lane, baseX };
      });

      const maxLeftDrift = 140;
      for (let lane = 0; lane < laneCount; lane++) {
        const idxs = layout.map((it, idx) => ({ it, idx })).filter(x => x.it.lane === lane).map(x => x.idx);
        for (let t = idxs.length - 2; t >= 0; t--) {
          const i0 = idxs[t];
          const i1 = idxs[t + 1];
          const targetMax = layout[i1].x - minGap;
          const driftMin = Math.max(xMin, layout[i0].baseX - maxLeftDrift);
          layout[i0].x = Math.max(driftMin, Math.min(layout[i0].x, targetMax));
        }
        for (let t = 1; t < idxs.length; t++) {
          const prev = layout[idxs[t - 1]];
          const cur = layout[idxs[t]];
          const minAllowed = prev.x + minGap;
          cur.x = Math.min(xMax, Math.max(cur.x, minAllowed));
        }
      }

      const maxY = (mapHeight != null && mapHeight > 0) ? (mapHeight - rectHeight - 4) : Infinity;
      if (Number.isFinite(maxY)) {
        layout.forEach(cell => {
          cell.y = Math.min(cell.y, maxY);
        });
      }

      lineGroup.selectAll(".timeline-rect")
        .data(selectedTimeData)
        .enter()
        .append("rect")
        .attr("class", "timeline-rect")
        .attr("x", (d, i) => layout[i].x - rectWidth / 2)
        .attr("y", (d, i) => layout[i].y)
        .attr("width", rectWidth)
        .attr("height", rectHeight)
        .attr("rx", 0)
        .attr("ry", 0)
        .attr("fill", "rgba(255,255,255,0.18)")
        .attr("stroke", "rgba(31,36,36,0.22)")
        .attr("stroke-width", 1)
        .attr("style", "pointer-events: visible")
        .on("click", (event, d) => {
          // 更新右侧卡片内容
          this.placeImpoDisplay = d.impo ? d.impo : "无";
          this.placeInfoDisplay = d.info ? d.info : "无";
        })
        .on("mouseover", function() {
          d3.select(this).transition().duration(300)
            .attr("fill", "rgba(179,58,43,0.10)")
            .attr("cursor", "pointer")
            .style("opacity", 0.8);
        })
        .on("mouseout", function() {
          d3.select(this).transition().duration(300)
            .attr("fill", "rgba(255,255,255,0.18)")
            .style("opacity", 1);
        });

      lineGroup.selectAll(".timeline-label-link")
        .data(selectedTimeData)
        .enter()
        .append("path")
        .attr("class", "timeline-label-link")
        .attr("d", (d, i) => {
          const y1 = centerY + timelineHeight / 2;
          const y2 = layout[i].y;
          const x1 = xScale(new Date(d.time));
          const x2 = layout[i].x;
          const yLabel = y2 + 10;
          const yMidBase = centerY + Math.round(70 * s);
          const yMid = yMidBase + (layout[i].lane === 0 ? 0 : Math.round(16 * s));
          return `M ${x1} ${y1} L ${x1} ${yMid} L ${x2} ${yMid} L ${x2} ${yLabel}`;
        })
        .attr("fill", "none")
        .attr("stroke", "rgba(31,36,36,0.22)")
        .attr("stroke-width", 1.2)
        .style("opacity", 0.9);

      lineGroup.selectAll(".timeline-text")
        .data(selectedTimeData)
        .enter()
        .append("text")
        .attr("class", "timeline-text")
        .attr("x", (d, i) => layout[i].x)
        .attr("y", (d, i) => layout[i].y)
        .attr("text-anchor", "middle")
        .attr("font-size", "12px")
        .attr("font-weight", "900")
        .attr("fill", "#000")
        .attr("style", "pointer-events: none")
        .each(function(d) {
          const textElement = d3.select(this);
          const characters = d.name.split("");
          const num = 6;
          const columnOffset = 12;
          const set2 = -2;
          const set3 = 1.5;
          const set4 = 5;
          const set5 = 8;
          const setX2 = 6;
          const setX3 = 12;
          const setX4 = 11;

          const totalCharacters = characters.length;
          let yOffset = 0;
          if (totalCharacters === 2) yOffset = set2;
          else if (totalCharacters === 3) yOffset = set3;
          else if (totalCharacters === 4) yOffset = set4;
          else if (totalCharacters >= 5) yOffset = set5;
          textElement.attr("y", +textElement.attr("y") + yOffset);

          const lineHeight = totalCharacters <= num ? 6 / totalCharacters : 1;
          const totalColumns = Math.ceil(totalCharacters / num);
          let xOffset = 0;
          if (totalColumns === 2) xOffset = setX2;
          else if (totalColumns === 3) xOffset = setX3;
          else if (totalColumns === 4) xOffset = setX4;
          textElement.attr("x", +textElement.attr("x") + xOffset);

          characters.forEach((char, index) => {
            const column = Math.floor(index / num);
            const row = index % num;
            textElement.append("tspan")
              .attr("x", +textElement.attr("x") - column * columnOffset)
              .attr("dy", row === 0 && column > 0 ? `-${(num - 1) * lineHeight}em` : lineHeight + "em")
              .text(char);
          });
        });
    },

    createTooltip(event, name) {
      const tooltipGroup = this.svg.select(".tooltip-group");
      if (tooltipGroup.empty()) this.svg.append("g").attr("class", "tooltip-group");
      const [x, y] = d3.pointer(event, this.svg.node());
      const boxW = 132, boxH = 44;
      const svgRect = this.svg.node().getBoundingClientRect();
      const safeX = Math.max(10, Math.min(x, svgRect.width - boxW - 10));
      const safeY = Math.max(10, Math.min(y, svgRect.height - boxH - 10));
      tooltipGroup.html(`
        <rect x="${safeX}" y="${safeY}" width="${boxW}" height="${boxH}" rx="10" fill="rgba(31,36,36,0.78)" stroke="rgba(246,241,226,0.55)" stroke-width="1"/>
        <text x="${safeX + boxW/2}" y="${safeY + 28}" text-anchor="middle" font-size="14" font-weight="800" fill="rgba(246,241,226,0.96)">${name}</text>
      `);
    },
    removeTooltip() {
      this.svg.select(".tooltip-group").html("");
    },

    handleResize() {
      this.resizeMap();
    },
  },
};
</script>

<style scoped>
/* 根层必须占满父级高度，否则 .map-container 的 height:100% 会为 0，地图无法初始化 */
.map-view {
  width: 100%;
  height: 100%;
  min-height: 0;
  display: flex;
  flex-direction: row;
  align-items: stretch;
  box-sizing: border-box;
}

.map-view .map-container {
  flex: 1;
  min-width: 0;
}

/* 详情：地图区域更宽，右侧信息栏收窄 */
.map-view.detail-mode .map-container {
  flex: 2.35;
  min-width: 0;
}
.map-view.detail-mode .story-cards {
  flex: 0.72;
  max-width: 32%;
  min-width: 180px;
}

/* 地图容器样式（原 .page-content 中的 .map-container 部分） */
.map-container {
  position: relative;
  margin: 0;
  width: 100%;
  height: 100%;
  min-height: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  background:
    radial-gradient(900px 520px at 55% 40%, rgba(31, 36, 36, 0.04), transparent 65%),
    linear-gradient(180deg, rgba(255, 255, 255, 0.54), rgba(255, 255, 255, 0.32)),
    linear-gradient(180deg, rgba(246, 241, 226, 0.92), rgba(239, 230, 207, 0.86));
  border: 1px solid var(--border);
  flex-shrink: 0;
}
.map-container::before {
  content: "";
  position: absolute;
  inset: 12px;
  border: 1px solid rgba(31, 36, 36, 0.16);
  pointer-events: none;
  opacity: 0.8;
}
.map-container svg {
  width: 100%;
  height: 100%;
  display: block;
}
.south-china-sea-image {
  position: absolute;
  top: 49%;
  left: 84%;
  transform: translate(-50%, -50%);
  width: 11%;
  max-width: 10vw;
  max-height: 15vw;
  height: auto;
  z-index: 2;
  opacity: 0.95;
  filter: drop-shadow(0 12px 20px rgba(0, 0, 0, 0.22)) saturate(0.95);
}

/* 右侧卡片：两栏固定各占 50% 高度，文字多在各自内层滚动 */
.story-cards {
  flex: 1;
  min-width: 0;
  min-height: 0;
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 10px;
  overflow: hidden;
  padding-right: 4px;
}
.card {
  flex: 1 1 0;
  min-height: 0;
  width: 100%;
  display: flex;
  flex-direction: column;
  background:
    radial-gradient(520px 260px at 30% 20%, rgba(179, 58, 43, 0.06), transparent 70%),
    linear-gradient(180deg, rgba(255, 255, 255, 0.54), rgba(255, 255, 255, 0.34)),
    linear-gradient(180deg, rgba(246, 241, 226, 0.92), rgba(239, 230, 207, 0.86));
  border: 1px solid var(--border);
  overflow: hidden;
  position: relative;
}
.card::before {
  content: "";
  position: absolute;
  inset: 12px;
  border: 1px dashed rgba(31, 36, 36, 0.16);
  pointer-events: none;
  opacity: 0.75;
}
.card-header {
  flex-shrink: 0;
  z-index: 1;
  padding: 10px 14px 8px;
  background: linear-gradient(180deg, rgba(246, 241, 226, 0.95), rgba(239, 230, 207, 0.72));
  border-bottom: 1px solid var(--hairline);
}
.hint {
  margin-top: 4px;
  font-size: 12px;
  color: var(--ink-2);
}
.inner-content {
  flex: 1 1 auto;
  min-height: 0;
  font-size: 16px;
  color: var(--ink-1);
  line-height: 1.85;
  margin: 8px 12px 10px;
  text-indent: 2em;
  overflow-y: auto;
  overflow-x: hidden;
}
.title {
  font-family: "KaiTi", "Kaiti SC", "STKaiti", "DFKai-SB", "Source Han Serif SC", serif;
  font-size: 20px;
  font-weight: 900;
  color: var(--ink-0);
  margin: 0;
  letter-spacing: 0.5px;
  position: relative;
  padding-left: 12px;
}
.title::before {
  content: "「";
  position: absolute;
  left: 0;
  top: 0;
  color: var(--seal);
  opacity: 0.9;
}
.title::after {
  content: "」";
  margin-left: 2px;
  color: var(--seal);
  opacity: 0.9;
}

/* 滚动条（仅卡片正文区） */
.inner-content::-webkit-scrollbar {
  width: 8px;
}
.inner-content::-webkit-scrollbar-thumb {
  background: linear-gradient(180deg, rgba(179, 58, 43, 0.34), rgba(89, 123, 122, 0.26));
  border-radius: 999px;
  border: 3px solid rgba(0, 0, 0, 0);
  background-clip: padding-box;
}
</style>