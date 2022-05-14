<template>
  <div class="list-container" id="list-container">
    <div
      v-for="(item, itemKey) in sortedItems"
      :key="itemKey"
      class="list-item"
      :id="'drag-element-' + itemKey"
      @mousedown.left="handleOnClickElement(itemKey, item.id)"
      @mouseenter="setElementDropPreview(itemKey)"
    >
      {{ item.text }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dragging: false,
      dropped: false,
      elementId: null,
      elementKey: null,
      elementClone: null,
      hoverElementKey: null,
      cursorX: null,
      cursorY: null,
      items: [
        {
          id: 0,
          text: "The very first item in the list.",
          order: 1,
          clone: false,
        },
        {
          id: 1,
          text: "The second item in the list wow.",
          order: 0,
          clone: false,
        },
        {
          id: 2,
          text: "The very 3rd item in the list.",
          order: 2,
          clone: false,
        },
        {
          id: 3,
          text: "The 4th item in the list wow.",
          order: 3,
          clone: false,
        },
        {
          id: 4,
          text: "The 5 item in the list.",
          order: 4,
          clone: false,
        },
        {
          id: 5,
          text: "The 6 in the list wow.",
          order: 5,
          clone: false,
        },
        {
          id: 6,
          text: "The 7777item in the list.",
          order: 6,
          clone: false,
        },
        {
          id: 7,
          text: "The 888888in the list wow.",
          order: 7,
          clone: false,
        },
      ],
    };
  },
  computed: {
    sortedItems: function () {
      function compare(a, b) {
        if (a.order < b.order) return -1;
        if (a.order > b.order) return 1;
        return 0;
      }

      let itemsCopy = this.items;

      return itemsCopy.sort(compare);
    },
  },
  methods: {
    onClickTestLog() {
      console.log("clicked");
    },
    handleOnClickElement(elementKey, elementId) {
      this.onClickTestLog();
      // Create elementClone
      let element = this.getElementByKey(elementKey);
      let elementClone = this.createElementClone(element);

      // Store copies of elementId and elementClone in data()
      this.elementId = elementId;
      this.elementClone = elementClone;
      this.elementKey = elementKey;

      // Configure element and elementClone
      let elementData = this.configureElementAndClone(element, elementClone);

      // Set handle drop listener
      this.handleDropListener(
        elementKey,
        elementData.elementWidth,
        elementData.elementHeight
      );

      setTimeout(() => {
        // User is now dragging
        this.handleOnHoldElement(
          element,
          elementClone,
          elementData.elementWidth,
          elementData.elementHeight
        );
      }, 50);
    },
    handleOnHoldElement(element, elementClone, elementWidth, elementHeight) {
      element.classList.remove("pre-drag");

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

        // Handle drag class
        elementClone.classList.add("dragging");
        this.dragging = true;
        this.handleMoveListener(elementClone, elementWidth, elementHeight);

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
    handleDropListener(elementKey, elementWidth, elementHeight) {
      let self = this;
      document.addEventListener("mouseup", function mouseUpHandler() {
        self.dropElement(elementKey, elementWidth, elementHeight);
        this.removeEventListener("mouseup", mouseUpHandler);
      });
    },
    setElementClonePosition(event, elementClone, elementWidth, elementHeight) {
      event.preventDefault();

      let elementCloneWidth = elementClone.offsetWidth;
      let elementCloneHeight = elementClone.offsetHeight;

      let cursorPositionTop = event.pageY;
      let cursorPositionLeft = event.pageX;

      // Store copy of cursor position in data
      this.cursorX = event.pageX;
      this.cursorY = event.pageY;

      // Calculate clone position
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
        if (hoverElementKey !== this.elementKey) {
          let domElement = this.getElementByKey(this.elementKey);
          let hoverDomElement = this.getElementByKey(hoverElementKey);
          let hoverDomParentElement = hoverDomElement.parentElement;
          // Save copy in data
          this.hoverElementKey = hoverElementKey;

          // Set
          this.setElementDropPreviewStyling(domElement);
          this.showElement(domElement);
          hoverDomParentElement.insertBefore(domElement, hoverDomElement);
        }
      }
    },
    setElementDropPreviewStyling(element) {
      element.style.opacity = 0.5;
      element.classList.add("shaking");
    },
    clearElementDropPreviewStyling(element) {
      element.style.opacity = 1;
      element.classList.remove("shaking");
    },
    dropElement(elementKey) {
      let element = this.getElementByKey(elementKey);

      this.showElement(element);
      this.clearElementDropPreviewStyling(element);

      this.dragging = false;

      let elementClone = this.getElementByKey(elementKey, true);

      if (elementClone) {
        elementClone.remove();
      } else {
        this.dropped = true;
      }

      this.elementId = null;
      this.elementClone = null;
      this.elementKey = null;
    },
    showElement(element) {
      element.style.display = "block";
    },
    hideElement(element) {
      element.style.display = "none";
    },
    createElementClone(element) {
      return element.cloneNode(true);
    },
    configureElementAndClone(element, clone) {
      element.classList.add("pre-drag");

      clone.id = clone.id + "-clone";
      clone.classList.remove("list-item");
      clone.classList.add("list-item-clone");

      let elementStylesCssText = this.getElementCssText(element);
      clone.style.cssText = elementStylesCssText;
      let elementPadding = this.getElementPadding(element);
      let elementPaddingY = elementPadding.top + elementPadding.bottom;
      let elementPaddingX = elementPadding.right + elementPadding.left;
      let elementWidth = element.clientWidth - elementPaddingX;
      let elementHeight = element.clientHeight - elementPaddingY;
      clone.style.width = elementWidth;
      clone.style.height = elementHeight;

      let elementData = {
        element: element,
        elementClone: clone,
        elementWidth: elementWidth,
        elementHeight: elementHeight,
      };

      return elementData;
    },
    getElementByKey(elementKey, clone = false) {
      if (clone) {
        return document.getElementById("drag-element-" + elementKey + "-clone");
      } else {
        return document.getElementById("drag-element-" + elementKey);
      }
    },
    getElementPadding(element) {
      let padding = {};

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
      const elementStyles = window.getComputedStyle(element);
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
  },
};
</script>

<style lang="scss" scoped>
$light-gray: #ccc;

.list-container {
  display: flex;
  flex-direction: column;
  max-width: 350px;
  margin-left: 500px;
  padding: 16px;
  border: 1px solid black;

  .list-item {
    border: 1px dashed $light-gray;
    padding: 32px;
    transition: opacity 0.1s ease-in;

    &:hover {
      cursor: grab;
    }

    &.pre-drag {
      opacity: 0.8;
    }

    &.shaking {
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

    &:not(:last-child) {
      margin-bottom: 16px;
    }
  }

  .list-item-clone {
    cursor: grabbing;

    &.dragging {
      animation: pulse 1s infinite !important;
      opacity: 0.8 !important;
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
  }
}
</style>
