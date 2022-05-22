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
      hiddenElementDisplayValue: null,
    };
  },
  mounted() {
    this.initialize();
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
        element.addEventListener("mouseenter", () => {
          this.setElementDropPreview(element.id);
        });
      }

      uniqueId = 0;
    },
    handleOnClick(elementId) {
      // Create elementClone
      let element = this.getElementById(elementId);
      let elementIndex = [].indexOf.call(element.parentNode.children, element);
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
    },
    handleOnHold(element, elementClone, elementWidth, elementHeight) {
      // Remove pre-drag class from element
      element.classList.remove(`${this.preDragElementClass}`);

      // Enter dragging state if user didn't drop item
      if (!this.dropped) {
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
        elementClone.classList.add("dragging");

        if (this.animateDragElement) {
          elementClone.classList.add("animated");
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
    },
    setElementDropPreview(hoverElementId) {
      if (this.dragging) {
        let domElement = this.getElementById(this.elementId);
        let elementSize = this.getElementSize(domElement);
        let elementWidth = elementSize.width;
        let elementHeight = elementSize.height;

        // Only set preview if user has moved cursor further than the size of the element the user is dragging over
        if (
          this.cursorMovedX > elementWidth ||
          this.cursorMovedY > elementHeight
        ) {
          let hoverDomElement = this.getElementById(hoverElementId);
          let hoverDomParentElement = hoverDomElement.parentElement;

          // Set element drop preview styling
          this.setElementDropPreviewStyling(domElement);

          // Show and insert drop preview element
          this.showElement(domElement);
          hoverDomParentElement.insertBefore(domElement, hoverDomElement);
        }
      }
    },
    setElementDropPreviewStyling(element) {
      element.classList.add("element-drop-preview");

      if (this.animateDropPreview) {
        element.classList.add("animated");
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
      this.$emit("elements-changed", {
        oldElementsArray: oldElementsArray,
        newElementsArray: newElementsArray,
        oldElementIndex: oldElementIndex,
        newElementIndex: newElementIndex,
        draggedElement: draggedElement,
      });
    },
    clearElementDropPreviewStyling(element) {
      // TODO: Implement drop preview styling customization
      element.style.opacity = "";
      element.classList.remove("element-drop-preview");

      if (this.animateDropPreview) {
        element.classList.remove("animated");
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
  },
};
</script>

<style lang="scss">
.dragging {
  pointer-events: none !important;

  &.animated {
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
  opacity: 0.5;

  &.animated {
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
