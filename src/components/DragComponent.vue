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
    dragItemClass: {
      type: String,
      required: false,
      default: "drag-item",
    },
    preDragItemClass: {
      type: String,
      required: false,
      default: "pre-drag",
    },
    dragDelay: {
      type: Number,
      required: false,
      default: 50,
    },
  },
  data() {
    return {
      dragging: false,
      dropped: false,
      elementKey: null,
      elementClone: null,
      cursorX: null,
      cursorY: null,
      cursorMovedX: 0,
      cursorMovedY: null,
      cursorMovedInitialized: false,
      hiddenElementDisplay: null,
      oldIndex: null,
      newIndex: null,
    };
  },
  mounted() {
    this.initializeElements();
  },
  methods: {
    updateElementValue(newValue) {
      if (Array.isArray(newValue)) {
        this.$emit("elements-changed", newValue);
      }
    },
    initializeElements() {
      // Set id on container
      let domElements = document.querySelectorAll(
        `#${this.dragContainerId} > *`
      );
      let elements = this.elements;

      for (let i = 0; i < domElements.length; i++) {
        let element = domElements[i];
        let item = elements[i];

        // Set id on elements
        element.id = `drag-element-${item.id}`;

        // Set event listeners
        element.addEventListener("mousedown", (e) => {
          if (e.which === 1) {
            this.handleOnClickElement(item.id, item.id);
          }
        });
        element.addEventListener("mouseenter", () => {
          this.setElementDropPreview(item.id);
        });
      }
    },
    handleOnClickElement(elementKey) {
      // Create elementClone
      let element = this.getElementByKey(elementKey);
      let elementIndex = [].indexOf.call(element.parentNode.children, element);
      let elementClone = this.createElementClone(element);

      // Store data
      this.elementKey = elementKey;
      this.oldIndex = elementIndex;
      this.elementClone = elementClone;

      // Configure element and elementClone
      let elementData = this.configureElementAndClone(element, elementClone);

      // Set handle drop listener
      this.handleDropListener(elementKey);

      // Enter dragging state after set amount of time (dragDelay prop)
      setTimeout(() => {
        this.handleOnHoldElement(
          element,
          elementClone,
          elementData.elementWidth,
          elementData.elementHeight
        );
      }, this.dragDelay);
    },
    handleOnHoldElement(element, elementClone, elementWidth, elementHeight) {
      // Remove pre-drag class from element
      element.classList.remove(`${this.preDragItemClass}`);

      // Enter dragging state if user didn't drop item
      if (!this.dropped) {
        // Replace element with elementClone
        let parentElement = element.parentNode;
        let elementIndexInParentContainer = Array.prototype.indexOf.call(
          parentElement.children,
          element
        );
        parentElement.insertBefore(
          elementClone,
          parentElement.children[elementIndexInParentContainer]
        );

        // Handle drag class/styling
        this.copyElementStylingToCloneDeep(element, elementClone);
        elementClone.classList.add("dragging");

        // Set move listener to position element clone at cursor position
        this.handleMoveListener(elementClone, elementWidth, elementHeight);

        // Set dragging true
        this.dragging = true;

        // Hide original element
        this.hideElement(element);
      } else {
        // Reset dropped status if user only clicked the element
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
    handleDropListener(elementKey) {
      let self = this;
      document.addEventListener("mouseup", function mouseUpHandler() {
        self.dropElement(elementKey);
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
    setElementDropPreview(hoverElementKey) {
      if (this.dragging) {
        let domElement = this.getElementByKey(this.elementKey);
        let elementSize = this.getElementSize(domElement);
        let elementWidth = elementSize.width;
        let elementHeight = elementSize.height;

        // Only set preview if user has moved cursor further than the size of the element the user is dragging over
        if (
          this.cursorMovedX > elementWidth ||
          this.cursorMovedY > elementHeight
        ) {
          let hoverDomElement = this.getElementByKey(hoverElementKey);
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
      // TODO: Implement drop preview styling customization
      element.style.opacity = 0.5;
      element.classList.add("shaking");
    },
    clearElementDropPreviewStyling(element) {
      // TODO: Implement drop preview styling customization
      element.style.opacity = 1;
      element.classList.remove("shaking");
    },
    dropElement(elementKey) {
      let element = this.getElementByKey(elementKey);

      // TODO: Handle items array order update
      // BUG: ON DROPPING ELEMENT, DOM ELEMENTS DO SOME FUNKY STUFF
      // #dev#

      let elementIndex = [].indexOf.call(element.parentNode.children, element);
      if (elementIndex > this.oldIndex) {
        elementIndex--;
      }
      this.newIndex = elementIndex;

      // Update array order
      let newElementsOrderArray = [...this.elements];

      let oldIndex = this.oldIndex;
      console.log("Old index: " + oldIndex);
      let newIndex = this.newIndex;
      console.log("New index: " + newIndex);

      // console.log(oldIndex, newIndex);

      if (newIndex > oldIndex || newIndex < oldIndex) {
        let elementToMove = newElementsOrderArray[oldIndex];
        // console.log(elementToMove);
        newElementsOrderArray.splice(oldIndex, 1);
        newElementsOrderArray.splice(newIndex, 0, elementToMove);
        console.table(this.elements);
        console.table(newElementsOrderArray);
      } else {
        console.log("order did not change");
      }

      // Emit updated elements array
      this.updateElementValue(newElementsOrderArray);

      // #dev#

      // Clean up
      this.clearElementDropPreviewStyling(element);
      this.dragging = false;
      this.elementKey = null;
      this.clearTrackedCursorMovement();
      let elementClone = this.getElementByKey(elementKey, true);

      if (elementClone) {
        elementClone.remove();
        this.elementClone = null;
      } else {
        // If user dropped item before it entered dragging state (time between user clicking + dragDelay prop)
        this.dropped = true;
      }

      this.showElement(element);
    },
    showElement(element) {
      element.style.display = this.hiddenElementDisplay;
      this.hiddenElementDisplay = null;
    },
    hideElement(element) {
      // Get element's display property value
      let elementDisplay = window.getComputedStyle(element).display;

      // Save copy of element's display property value to be able to set it back later in showElement
      if (elementDisplay) {
        this.hiddenElementDisplay = elementDisplay;
      } else {
        // If not set, assume value is block
        this.hiddenElementDisplay = "block";
      }

      // Hide element
      element.style.display = "none";
    },
    createElementClone(element) {
      return element.cloneNode(true);
    },
    configureElementAndClone(element, clone) {
      element.classList.add(`${this.preDragItemClass}`);
      clone.id = clone.id + "-clone";
      clone.classList.remove(this.dragItemClass);
      clone.classList.add(`${this.dragItemClass}-clone`);

      // Set clone's size
      let elementPadding = this.getElementPadding(element);
      let elementPaddingY = elementPadding.top + elementPadding.bottom;
      let elementPaddingX = elementPadding.right + elementPadding.left;
      let elementWidth = element.clientWidth - elementPaddingX;
      let elementHeight = element.clientHeight - elementPaddingY;
      clone.style.width = elementWidth;
      clone.style.height = elementHeight;

      // Return
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
        `#${this.dragContainerId} > #${this.getElementByKey(
          this.elementKey,
          false,
          true
        )}  *`
      );

      let cloneChildNodes = document.querySelectorAll(
        `#${this.dragContainerId} > #${this.getElementByKey(
          this.elementKey,
          true,
          true
        )} *`
      );

      // Copy element's children styles to clone's children
      for (let i = 0; i < elementChildNodes.length; i++) {
        let elementChildNode = elementChildNodes[i];
        let cloneChildNode = cloneChildNodes[i];

        let elementChildNodeStylesCssText =
          this.getElementCssText(elementChildNode);
        cloneChildNode.style.cssText = elementChildNodeStylesCssText;
        cloneChildNode.style.pointerEvents = "none";
      }
    },
    getElementByKey(elementKey, clone = false, string = false) {
      if (clone) {
        if (string) {
          return "drag-element-" + elementKey + "-clone";
        } else {
          return document.getElementById(
            "drag-element-" + elementKey + "-clone"
          );
        }
      } else {
        if (string) {
          return "drag-element-" + elementKey;
        } else {
          return document.getElementById("drag-element-" + elementKey);
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
    clearTrackedCursorMovement() {
      this.cursorMovedInitialized = false;
      this.cursorX = null;
      this.cursorY = null;
      this.cursorMovedX = null;
      this.cursorMovedY = null;
    },
  },
};
</script>

<style lang="scss">
.dragging {
  animation: pulse 1s infinite !important;
  opacity: 0.6 !important;
  pointer-events: none !important;
  // padding: 0;

  @keyframes pulse {
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

.shaking {
  animation: shake-with-pause 2s infinite !important;

  @keyframes shake-with-pause {
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
</style>
