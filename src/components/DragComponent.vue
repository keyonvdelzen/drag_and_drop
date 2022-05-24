<template>
  <slot></slot>
</template>

<script>
export default {
  emits: ["elements-changed"],
  props: {
    elements: {
      type: Array,
      required: true,
    },
    orientation: {
      type: String,
      required: true,
      default: "vertical",
      validator(value) {
        // The value must match one of these strings
        return ["vertical", "horizontal"].includes(value);
      },
    },
    dragContainerId: {
      type: String,
      required: false,
      default: "drag-container",
    },
    dragElementClass: {
      type: String,
      required: false,
      default: "drag-element",
    },
    preDragElementClass: {
      type: String,
      required: false,
      default: "pre-drag",
    },
    dragDelay: {
      type: Number,
      required: false,
      default: 150,
    },
    animateDropPreview: {
      type: Boolean,
      required: false,
      default: false,
    },
    animateDragElement: {
      type: Boolean,
      required: false,
      default: false,
    },
    verbose: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      dragging: false,
      dropped: false,
      elementId: null,
      elementClone: null,
      oldIndex: null,
      newIndex: null,
      cursorX: null,
      cursorY: null,
      cursorMovedX: 0,
      cursorMovedY: 0,
      cursorMovedInitialized: false,
      setElementDropPreviewIsUnlocked: false,
      hiddenElementDisplayValue: null,
      travelDistanceWatcherSwitches: {
        // Dev
        thousand: true,
        tenThousand: true,
        hundredThousand: true,
        million: true,
      },
    };
  },
  mounted() {
    this.initialize();
  },
  computed: {
    consoleStyles() {
      let groupStyles = [
        "margin-bottom: 3px",
        "border-radius: 4px",
        "padding: 4px 8px",
        "font-weight: bold",
        "color: white",
        "background: gray",
      ].join(";");

      let timeStyles = [
        "margin-right: 8px",
        "margin-bottom: 3px",
        "border-radius: 4px",
        "padding: 4px 8px",
        "font-weight: bold",
        "color: white",
        "background: gray",
      ].join(";");

      let infoStylesDefault = [
        "margin-right: 8px",
        "margin-bottom: 3px",
        "border-radius: 4px",
        "padding: 4px 8px",
        "font-weight: bold",
        "color: white",
        "background: #4a4a4a",
      ].join(";");

      let infoStylesSuccess = [
        "margin-right: 8px",
        "margin-bottom: 4px",
        "border-radius: 4px",
        "padding: 4px 8px",
        "font-weight: bold",
        "color: white",
        "background: #48c78e",
      ].join(";");

      let infoStylesWarning = [
        "margin-right: 8px",
        "margin-bottom: 4px",
        "border-radius: 4px",
        "padding: 4px 8px",
        "font-weight: bold",
        "color: rgba(0,0,0,.7)",
        "background: #ffe08a",
      ].join(";");

      let infoStylesMagic = [
        "margin-right: 8px",
        "margin-bottom: 4px",
        "border-radius: 4px",
        "padding: 4px 8px",
        "font-weight: bold",
        "color: rgba(0,0,0,.7)",
        "background: #FFC0CB",
      ].join(";");

      let styles = {
        group: groupStyles,
        time: timeStyles,
        info: {
          default: infoStylesDefault,
          success: infoStylesSuccess,
          warning: infoStylesWarning,
          magic: infoStylesMagic,
        },
      };

      return styles;
    },
    // Dev
    cursorDistanceTraveled() {
      return this.cursorMovedX + this.cursorMovedY;
    },
  },
  methods: {
    initialize() {
      // Get all direct child elements of element with containerId
      let domElements = document.querySelectorAll(
        `#${this.dragContainerId} > *`
      );

      let uniqueId = 0;

      // Set unique ids on child elements and bind event listeners
      for (let i = 0; i < domElements.length; i++) {
        let element = domElements[i];

        // Set unique id on element (if element has no id yet)
        if (element.id.length === 0) {
          element.id = `drag-element-${uniqueId}`;
          uniqueId++;
        }

        // Bind event listeners
        element.addEventListener("mousedown", (e) => {
          if (e.which === 1) {
            this.handleOnClick(element.id);
          }
        });
        element.addEventListener("mouseenter", (e) => {
          this.setElementDropPreview(e, element.id);
        });
        element.addEventListener("mouseout", (e) => {
          this.setElementDropPreview(e, element.id);
        });
      }

      uniqueId = 0;
    },
    handleOnClick(elementId) {
      if (!this.dropped) {
        if (this.verbose) {
          this.logEnteredClickState();
        }

        // Create elementClone
        let element = this.getElementById(elementId);
        let elementIndex = [].indexOf.call(
          element.parentNode.children,
          element
        );
        let elementClone = this.createElementClone(element);

        // Call drop listener handler
        this.handleDropListener(elementId);

        // Store data
        this.elementId = elementId;
        this.oldIndex = elementIndex;
        this.elementClone = elementClone;

        // Configure element and elementClone
        let elementData = this.configureElementAndClone(element, elementClone);

        // Enter dragging state after x amount of time (customizable)
        setTimeout(() => {
          this.handleOnHold(
            element,
            elementClone,
            elementData.elementWidth,
            elementData.elementHeight
          );
        }, this.dragDelay);
      }
    },
    handleOnHold(element, elementClone, elementWidth, elementHeight) {
      // Remove pre-drag class from element
      element.classList.remove(`${this.preDragElementClass}`);

      // Enter dragging state if user didn't drop item
      if (!this.dropped) {
        if (this.verbose) {
          this.logEnteredDragState();
        }

        // Insert elementClone
        let parentElement = element.parentNode;
        let elementIndexInParentContainer = Array.prototype.indexOf.call(
          parentElement.children,
          element
        );
        parentElement.insertBefore(
          elementClone,
          parentElement.children[elementIndexInParentContainer]
        );

        // Copy element styling to elementClone
        this.copyElementStylingToCloneDeep(element, elementClone);

        // Set dragging class on elementClone
        elementClone.classList.add("element-is-dragging");

        if (this.animateDragElement) {
          elementClone.classList.add("animated-clone-pulse");
        }

        // Set dragging true
        this.dragging = true;

        // Bind move listener to position element clone at cursor position
        this.handleMoveListener(elementClone, elementWidth, elementHeight);

        // Hide original element
        this.hideElement(element);
      } else {
        // Reset dropped status if user dropped the element before entering dragging state
        this.dropped = false;

        if (this.verbose) {
          this.logDroppedElement();
        }
      }
    },
    handleMoveListener(elementClone, elementWidth, elementHeight) {
      let self = this;
      document.addEventListener("mousemove", function mouseMoveHandler(event) {
        if (self.dragging) {
          self.setElementClonePosition(
            event,
            elementClone,
            elementWidth,
            elementHeight
          );
        } else {
          this.removeEventListener("mousemove", mouseMoveHandler);
        }
      });
    },
    handleDropListener(elementId) {
      let self = this;
      document.addEventListener("mouseup", function mouseUpHandler() {
        self.dropElement(elementId);
        this.removeEventListener("mouseup", mouseUpHandler);
      });
    },
    setElementClonePosition(event, elementClone, elementWidth, elementHeight) {
      event.preventDefault();

      let cursorPositionLeft = event.pageX;
      let cursorPositionTop = event.pageY;
      let elementCloneWidth = elementClone.offsetWidth;
      let elementCloneHeight = elementClone.offsetHeight;

      // Call this before updating cursorX & cursorY
      this.trackCursorMovement(
        this.cursorX,
        this.cursorY,
        cursorPositionLeft,
        cursorPositionTop
      );

      // Store copy of cursor position in data
      this.cursorX = cursorPositionLeft;
      this.cursorY = cursorPositionTop;

      // Calculate clone position centered at cursor position
      let elementClonePositionTop = cursorPositionTop - elementCloneHeight / 2;
      let elementClonePositionLeft = cursorPositionLeft - elementCloneWidth / 2;

      // Set clone position, size and visibility
      elementClone.style.top = elementClonePositionTop + "px";
      elementClone.style.left = elementClonePositionLeft + "px";
      elementClone.style.width = elementWidth + "px";
      elementClone.style.height = elementHeight + "px";
      elementClone.style.visibility = "visible";
      elementClone.style.position = "absolute";

      // eslint-disable-next-line
      // debugger;
    },
    setElementDropPreview(event, hoverElementId) {
      if (this.dragging) {
        // Get data
        let domElement = this.getElementById(this.elementId);
        let hoverDomElement = this.getElementById(hoverElementId);
        let hoverElementSize = this.getElementSize(hoverDomElement);
        let hoverElementWidth = hoverElementSize.width;
        let hoverElementHeight = hoverElementSize.height;

        // Only set preview if user has moved cursor further than the size of the element the user is dragging over
        // or if user has already set a drop preview in current drag
        // Given a 0.5 factor correction to smooth out the movement
        let movedFarEnoughX = hoverElementWidth * 0.5 < this.cursorMovedX;
        let movedFarEnoughY = hoverElementHeight * 0.5 < this.cursorMovedY;

        if (
          (this.orientation === "horizontal" && movedFarEnoughX) ||
          (this.orientation === "vertical" && movedFarEnoughY) ||
          this.setElementDropPreviewIsUnlocked
        ) {
          this.setElementDropPreviewIsUnlocked = true;

          let hoverDomParentElement = hoverDomElement.parentElement;

          // Set element drop preview styling
          this.setElementDropPreviewStyling(domElement);

          // Get cursor enter/out position relative to element to calculate whether to insert before or after element
          let rect = event.target.getBoundingClientRect();
          let relativeCursorPositionX = event.clientX - rect.left;
          let relativeCursorPositionY = event.clientY - rect.top;
          let halfWidth = hoverDomElement.offsetWidth / 2;
          let halfHeight = hoverDomElement.offsetHeight / 2;

          // Show and insert drop preview element
          if (this.orientation === "vertical") {
            // If mouse enter/left on top half of element
            if (relativeCursorPositionY < halfHeight) {
              hoverDomParentElement.insertBefore(domElement, hoverDomElement);
            }
            // If mouse enter/left on bottom half of element
            else {
              hoverDomParentElement.insertBefore(
                domElement,
                hoverDomElement.nextSibling
              );
            }
          } else {
            // If mouse enter/left on left half of element
            if (relativeCursorPositionX < halfWidth) {
              hoverDomParentElement.insertBefore(domElement, hoverDomElement);
            }
            // If mouse enter/left on right half of element
            else {
              hoverDomParentElement.insertBefore(
                domElement,
                hoverDomElement.nextSibling
              );
            }
          }
          this.showElement(domElement);
        }
      }
    },
    setElementDropPreviewStyling(element) {
      element.classList.add("element-drop-preview");

      if (this.animateDropPreview) {
        element.classList.add("animated-shake-with-pause");
      }
    },
    showElement(element) {
      element.style.display = this.hiddenElementDisplayValue;
      this.hiddenElementDisplayValue = null;
    },
    hideElement(element) {
      // Get element's display property value
      let elementDisplay = window.getComputedStyle(element).display;

      // Save copy of element's display property value to be able to set it back later in this.showElement()
      if (elementDisplay) {
        this.hiddenElementDisplayValue = elementDisplay;
      } else {
        // If not set, assume value is block
        this.hiddenElementDisplayValue = "block";
      }

      // Hide element
      element.style.display = "none";
    },
    createElementClone(element) {
      return element.cloneNode(true);
    },
    configureElementAndClone(element, clone) {
      // Set pre-drag class on element
      element.classList.add(`${this.preDragElementClass}`);

      // Set clone's id
      clone.id = clone.id + "-clone";

      // Set clone's size
      let elementPadding = this.getElementPadding(element);
      let elementPaddingY = elementPadding.top + elementPadding.bottom;
      let elementPaddingX = elementPadding.right + elementPadding.left;
      let elementWidth = element.clientWidth - elementPaddingX;
      let elementHeight = element.clientHeight - elementPaddingY;
      clone.style.width = elementWidth;
      clone.style.height = elementHeight;

      // Return data
      let elementData = {
        element: element,
        elementClone: clone,
        elementWidth: elementWidth,
        elementHeight: elementHeight,
      };

      return elementData;
    },
    copyElementStylingToCloneDeep(element, clone) {
      // Copy element's style to clone's style
      let elementStylesCssText = this.getElementCssText(element);
      clone.style.cssText = elementStylesCssText;
      clone.style.pointerEvents = "none";
      clone.style.opacity = element.style.opacity; // Because somehow opacity gets copied over from pre-drag class..?

      // Get element's and clone's child nodes
      let elementChildNodes = document.querySelectorAll(
        `#${this.dragContainerId} > #${this.getElementById(
          this.elementId,
          false,
          true
        )}  *`
      );

      let cloneChildNodes = document.querySelectorAll(
        `#${this.dragContainerId} > #${this.getElementById(
          this.elementId,
          true,
          true
        )} *`
      );

      // Copy element's children's styling to clone's children
      for (let i = 0; i < elementChildNodes.length; i++) {
        let elementChildNode = elementChildNodes[i];
        let cloneChildNode = cloneChildNodes[i];

        let elementChildNodeStylesCssText =
          this.getElementCssText(elementChildNode);
        cloneChildNode.style.cssText = elementChildNodeStylesCssText;

        // Set pointer events to none,
        // because else the elementClone prevents the hover event of the hoverElement from being triggered
        // TODO: find a way to set the cursor to 'dragging' when dragging
        cloneChildNode.style.pointerEvents = "none";
      }
    },
    trackCursorMovement(
      oldPositionX,
      oldPositionY,
      newpositionX,
      newPositionY
    ) {
      // Calculates cumulative cursor movement and stores it in data()
      let cursorXDifference = 0;
      let cursorYDifference = 0;

      if (this.cursorMovedInitialized) {
        cursorXDifference = newpositionX - oldPositionX;
        cursorYDifference = newPositionY - oldPositionY;
      } else {
        this.cursorMovedInitialized = true;
      }

      this.cursorMovedX = this.cursorMovedX + Math.abs(cursorXDifference);
      this.cursorMovedY = this.cursorMovedY + Math.abs(cursorYDifference);
    },
    dropElement(elementId) {
      let element = this.getElementById(elementId);

      // Get and set element index
      let elementIndex = [].indexOf.call(element.parentNode.children, element);

      if (elementIndex > this.oldIndex) {
        elementIndex--;
      }

      this.newIndex = elementIndex;

      let oldIndex = this.oldIndex;
      let newIndex = this.newIndex;

      // Update array order if oldIndex !== newIndex
      let newElementsArray = [...this.elements];
      if (newIndex > oldIndex || newIndex < oldIndex) {
        let elementToMove = newElementsArray[oldIndex];
        newElementsArray.splice(oldIndex, 1);
        newElementsArray.splice(newIndex, 0, elementToMove);

        // Emit updated elements array
        let oldElementsArray = this.elements;
        this.emitElementsChanged(
          oldElementsArray,
          newElementsArray,
          oldIndex,
          newIndex,
          element
        );
      }

      // Clean up state
      this.cleanUpState(element, elementId);
    },
    emitElementsChanged(
      oldElementsArray,
      newElementsArray,
      oldElementIndex,
      newElementIndex,
      draggedElement
    ) {
      let data = {
        oldElementsArray: oldElementsArray,
        newElementsArray: newElementsArray,
        oldElementIndex: oldElementIndex,
        newElementIndex: newElementIndex,
        draggedElement: draggedElement,
      };

      this.$emit("elements-changed", data);

      if (this.verbose) {
        this.logMovedElement(data);
      }
    },
    clearElementDropPreviewStyling(element) {
      // TODO: Implement drop preview styling customization
      element.style.opacity = "";
      element.classList.remove("element-drop-preview");

      if (this.animateDropPreview) {
        element.classList.remove("animated-shake-with-pause");
      }
    },
    cleanUpState(element, elementId) {
      let elementClone = this.getElementById(elementId, true);

      if (elementClone) {
        elementClone.remove();
      } else {
        // If user dropped item before it entered dragging state (time between user clicking + dragDelay prop)
        this.dropped = true;
      }

      this.clearElementDropPreviewStyling(element);

      this.dragging = false;
      this.elementId = null;
      this.elementClone = null;
      this.oldIndex = null;
      this.newIndex = null;
      this.cursorX = null;
      this.cursorY = null;
      this.cursorMovedX = 0;
      this.cursorMovedY = 0;
      this.cursorMovedInitialized = false;
      this.setElementDropPreviewIsUnlocked = false;

      this.showElement(element);
    },
    getElementById(elementId, clone = false, string = false) {
      if (clone) {
        if (string) {
          return elementId + "-clone";
        } else {
          return document.getElementById(elementId + "-clone");
        }
      } else {
        if (string) {
          return elementId;
        } else {
          return document.getElementById(elementId);
        }
      }
    },
    getElementPadding(element) {
      let padding = {
        top: null,
        right: null,
        bottom: null,
        left: null,
      };

      padding.top = parseFloat(
        window.getComputedStyle(element, null).getPropertyValue("padding-top")
      );
      padding.right = parseFloat(
        window.getComputedStyle(element, null).getPropertyValue("padding-right")
      );
      padding.bottom = parseFloat(
        window
          .getComputedStyle(element, null)
          .getPropertyValue("padding-bottom")
      );
      padding.left = parseFloat(
        window.getComputedStyle(element, null).getPropertyValue("padding-left")
      );

      return padding;
    },
    getElementCssText(element) {
      let elementStyles = window.getComputedStyle(element);
      let elementStylesCssText = elementStyles.cssText;

      if (!elementStylesCssText) {
        elementStylesCssText = Array.from(elementStyles).reduce(
          (str, property) => {
            return `${str}${property}:${elementStyles.getPropertyValue(
              property
            )};`;
          },
          ""
        );
      }

      return elementStylesCssText;
    },
    getElementSize(element) {
      let elementSize = {
        width: null,
        height: null,
      };

      let elementWidth = element.clientWidth;
      let elementHeight = element.clientHeight;
      elementSize.width = elementWidth;
      elementSize.height = elementHeight;

      return elementSize;
    },
    logEnteredClickState() {
      console.log(
        "%c⌛ " + this.getTimeFormatted() + "%cEntered click state",
        this.consoleStyles.time,
        this.consoleStyles.info.default
      );
    },
    logDroppedElement() {
      console.log(
        "%c⌛ " +
          this.getTimeFormatted() +
          "%cDropped element before entering drag state",
        this.consoleStyles.time,
        this.consoleStyles.info.warning
      );
    },
    logEnteredDragState() {
      console.log(
        "%c⌛ " + this.getTimeFormatted() + "%cEntered drag state",
        this.consoleStyles.time,
        this.consoleStyles.info.default
      );
    },
    logMovedElement(data) {
      console.groupCollapsed(
        "%c⌛ " + this.getTimeFormatted() + "%cMoved element 🚀",
        this.consoleStyles.time,
        this.consoleStyles.info.success
      );

      // Moved element
      console.groupCollapsed(
        "%cMoved element",
        this.consoleStyles.info.default
      );
      console.log(data.draggedElement);
      console.groupEnd();

      // Old index
      console.log(
        "%cOld index: " + data.oldElementIndex,
        this.consoleStyles.info.default
      );

      // New index
      console.log(
        "%cNew index: " + data.newElementIndex,
        this.consoleStyles.info.default
      );

      // Old array
      console.groupCollapsed("%cOld array", this.consoleStyles.info.default);
      console.table(data.oldElementsArray);
      console.groupEnd();

      // New array
      console.groupCollapsed("%cNew array", this.consoleStyles.info.default);
      console.table(data.newElementsArray);
      console.groupEnd();

      console.groupEnd();
    },
    getTimeFormatted() {
      let time = new Date();
      time =
        String(time.getHours()).padStart(2, "0") +
        ":" +
        String(time.getMinutes()).padStart(2, "0") +
        ":" +
        String(time.getSeconds()).padStart(2, "0") +
        ":" +
        String(time.getMilliseconds()).padStart(3, "0");
      return time;
    },
  },
  watch: {
    // Dev
    cursorDistanceTraveled(d) {
      if (this.travelDistanceWatcherSwitches.thousand) {
        if (d > 1000) {
          if (this.verbose) {
            console.log(
              "%c🏁" + "%cTraveled 1k pixels",
              this.consoleStyles.time,
              this.consoleStyles.info.magic
            );
          }
          this.travelDistanceWatcherSwitches.thousand = false;
        }
      } else if (this.travelDistanceWatcherSwitches.tenThousand) {
        if (d > 10000) {
          if (this.verbose) {
            console.log(
              "%c🏁" + "%cTraveled 10k pixels",
              this.consoleStyles.time,
              this.consoleStyles.info.magic
            );
          }
          this.travelDistanceWatcherSwitches.tenThousand = false;
        }
      } else if (this.travelDistanceWatcherSwitches.hundredThousand) {
        if (d > 100000) {
          if (this.verbose) {
            console.log(
              "%c🏁" + "%cTraveled 100k pixels",
              this.consoleStyles.time,
              this.consoleStyles.info.magic
            );
          }
          this.travelDistanceWatcherSwitches.hundredThousand = false;
        }
      } else if (this.travelDistanceWatcherSwitches.million) {
        if (d > 1000000) {
          if (this.verbose) {
            console.log(
              "%c🏁" + "%cTraveled one million pixels!",
              this.consoleStyles.time,
              this.consoleStyles.info.magic
            );
          }
          this.travelDistanceWatcherSwitches.million = false;
        }
      }
    },
  },
};
</script>

<style lang="scss">
.element-is-dragging {
  z-index: 100000000000;

  &.animated-clone-pulse {
    animation: element-clone-pulse 1s infinite !important;

    @keyframes element-clone-pulse {
      0% {
        transform: scale(0.95);
        box-shadow: 0 0 0 0 rgba(0, 0, 0, 0.2);
      }

      70% {
        transform: scale(1);
        box-shadow: 0 0 0 5px rgba(0, 0, 0, 0);
      }

      100% {
        transform: scale(0.95);
        box-shadow: 0 0 0 0 rgba(0, 0, 0, 0);
      }
    }
  }
}

.element-drop-preview {
  z-index: -100000000000;
  opacity: 0.5 !important;

  &.animated-shake-with-pause {
    animation: element-drop-preview-shake-with-pause 2s infinite !important;

    @keyframes element-drop-preview-shake-with-pause {
      0% {
        transform: translate(1px, 1px) rotate(0deg);
      }
      5% {
        transform: translate(-1px, -2px) rotate(-1deg);
      }
      10% {
        transform: translate(-3px, 0px) rotate(1deg);
      }
      15% {
        transform: translate(3px, 2px) rotate(0deg);
      }
      20% {
        transform: translate(1px, -1px) rotate(1deg);
      }
      25% {
        transform: translate(-1px, 2px) rotate(-1deg);
      }
      30% {
        transform: translate(-3px, 1px) rotate(0deg);
      }
      35% {
        transform: translate(3px, 1px) rotate(-1deg);
      }
      40% {
        transform: translate(-1px, -1px) rotate(1deg);
      }
      45% {
        transform: translate(1px, 2px) rotate(0deg);
      }
      50% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      55% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      60% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      65% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      70% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      75% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      80% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      85% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      90% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      95% {
        transform: translate(0px, 0px) rotate(0deg);
      }
      100% {
        transform: translate(0px, 0px) rotate(0deg);
      }
    }
  }
}
</style>
