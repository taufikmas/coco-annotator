<script>
import paper from "paper";
import tool from "@/mixins/toolBar/tool";

export default {
  name: "EraserTool",
  mixins: [tool],
  props: {
    scale: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      icon: "fa-paint-brush",
      name: "Brush",
      cursor: "none",
      scaleFactor: 3,
      brush: {
        path: null,
        pathOptions: {
          strokeColor: "white",
          strokeWidth: 1,
          radius: 30
        }
      }
    };
  },
  methods: {
    removeBrush() {
      if (this.brush.path != null) {
        this.brush.path.remove();
        this.brush.path = null;
      }
    },
    moveBrush(point) {
      if (this.brush.path == null) this.createBrush();

      this.brush.path.bringToFront();
      this.brush.path.position = point;
    },
    createBrush(center) {
      center = center || new paper.Point(0, 0);

      this.brush.path = new paper.Path.Circle({
        strokeColor: this.brush.pathOptions.strokeColor,
        strokeWidth: this.brush.pathOptions.strokeWidth,
        radius: this.brush.pathOptions.radius,
        center: center
      });
    },
    onMouseMove(event) {
      this.moveBrush(event.point);
    },
    onMouseDown() {
      this.$parent.currentAnnotation.createUndoAction("Unite");
      this.draw();
    },
    onMouseUp() {
      this.$parent.currentAnnotation.simplifyPath();
    },
    onMouseDrag(event) {
      this.moveBrush(event.point);
      this.draw();
    },
    /**
     * Unites current brush with selected annotation
     */
    draw() {
      // Undo action, will be handled on mouse down
      // Simplify, will be handled on mouse up
      this.$parent.currentAnnotation.unite(this.brush.path, false, false);
    },
    decreaseRadius() {
      if (!this.isActive) return;
      this.brush.pathOptions.radius -= 2;
    },
    increaseRadius() {
      if (!this.isActive) return;
      this.brush.pathOptions.radius += 2;
    },
    export() {
      return {
        strokeColor: this.brush.pathOptions.strokeColor,
        radius: this.brush.pathOptions.radius
      };
    },
    setPreferences(pref) {
      this.brush.pathOptions.strokeColor =
        pref.strokeColor || this.brush.pathOptions.strokeColor;
      this.brush.pathOptions.radius =
        pref.radius || this.brush.pathOptions.radius;
    }
  },
  computed: {
    isDisabled() {
      return this.$parent.current.annotation == -1;
    }
  },
  watch: {
    "brush.pathOptions.radius"() {
      if (this.brush.path == null) return;

      let position = this.brush.path.position;
      this.brush.path.remove();
      this.createBrush(position);
    },
    "brush.pathOptions.strokeColor"(newColor) {
      if (this.brush.path == null) return;

      this.brush.path.strokeColor = newColor;
    },
    isActive(active) {
      if (this.brush.path != null) {
        this.brush.path.visible = active;
      }

      if (active) {
        this.tool.activate();
      }
    },
    /**
     * Change width of stroke based on zoom of image
     */
    scale(newScale) {
      this.brush.pathOptions.strokeWidth = newScale * this.scaleFactor;
      if (this.brush.path != null)
        this.brush.path.strokeWidth = newScale * this.scaleFactor;
    }
  },
  mounted() {}
};
</script>
