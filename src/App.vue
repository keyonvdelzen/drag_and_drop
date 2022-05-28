<template>
  <div>
    <div class="controls">
      <button
        @click="verbose = !verbose"
        class="button"
        :class="{ active: verbose }"
      >
        Verbose
      </button>
    </div>

    <div class="lists-container">
      <!-- Vertical -->
      <drag-component
        @elements-changed="handleElementsChanged"
        :elements="itemsVertical"
        orientation="vertical"
        dragContainerId="drag-container-vertical"
        dragElementClass="drag-element"
        preDragElementClass="pre-drag"
        :dragDelay="150"
        :animateDragElement="true"
        :animateDropPreview="true"
        :verbose="verbose"
      >
        <div class="drag-container-vertical" id="drag-container-vertical">
          <div
            v-for="item in itemsVertical"
            :key="item.id"
            :id="'drag-element-' + item.id"
            @click.prevent="handleElementClick"
            class="drag-element"
            :style="`background-color: ${item.color}`"
          >
            <div class="content">
              <div class="text">{{ item.text }}</div>
            </div>
          </div>
        </div>
      </drag-component>
      <!-- Horizontal -->
      <drag-component
        @elements-changed="handleElementsChanged"
        :elements="itemsHorizontal"
        orientation="horizontal"
        :dragDelay="150"
        dragContainerId="drag-container-horizontal"
        dragElementClass="drag-element"
        preDragElementClass="pre-drag"
        :animateDragElement="true"
        :animateDropPreview="true"
        :verbose="verbose"
      >
        <div class="drag-container-horizontal" id="drag-container-horizontal">
          <div
            v-for="item in itemsHorizontal"
            :key="item.id"
            :id="'drag-element-' + item.id"
            @click.prevent="handleElementClick"
            class="drag-element"
            :style="`background-color: ${item.color}`"
          >
            <div class="content">
              <div class="text">{{ item.text }}</div>
            </div>
          </div>
        </div>
      </drag-component>
    </div>
  </div>
</template>

<script>
import DragComponent from "./components/DragComponent.vue";

export default {
  name: "App",
  components: {
    DragComponent,
  },
  data() {
    return {
      verbose: false,
      itemsVertical: [
        {
          id: 0,
          text: "condimentum id venenatis a condimentum vitae sapien pellentesque habitant morbi tristique senectus et",
          order: 1,
          color: "lightpink",
        },
        {
          id: 1,
          text: "ultricies tristique nulla aliquet enim tortor at auctor urna nunc id cursus metus aliquam eleifend mi in",
          order: 0,
          color: "lightcyan",
        },
        {
          id: 2,
          text: "at in tellus integer feugiat scelerisque varius morbi enim nunc faucibus a pellentesque sit amet porttitor dolor morbi non arcu risus quis varius quam quisque id diam vel quam elementum",
          order: 3,
          color: "linen",
        },
        {
          id: 3,
          text: "egestas maecenas pharetra convallis posuere morbi leo urna molestie at elementum eu facilisis sed odio morbi quis commodo odio aenean sed adipiscing diam donec adipiscing tristique risus nec feugiat in fermentum",
          order: 2,
          color: "lightblue",
        },
      ],
      itemsHorizontal: [
        {
          id: 4,
          text: "eros donec ac odio tempor orci dapibus ultrices in  eu consequat ac felis donec et odio pellentesque diam volutpat commodo sed egestas egestas",
          order: 4,
          color: "lightgreen",
        },
        {
          id: 5,
          text: "tellus pellentesque eu tincidunt tortor aliquam nulla facilisi cras fermentum odio eu feugiat pretium nibh ipsum consequat nisl vel pretium lectus quam id leo in vitae turpis massa sed elementum tempus",
          order: 6,
          color: "lightgray",
        },
        {
          id: 6,
          text: "magna sit amet volutpat consequat mauris nunc congue nisi vitae suscipit tellus mauris a diam",
          order: 5,
          color: "lavender",
        },
        {
          id: 7,
          text: "mi tempus imperdiet nulla malesuada pellentesque cum sociis natoque penatibus et magnis dis parturient montes nascetur ridiculus mus mauris vitae ultricies leo integer malesuada nunc vel risus commodo",
          order: 7,
          color: "lemonchiffon",
        },
      ],
    };
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

      let styles = {
        group: groupStyles,
        time: timeStyles,
        info: {
          default: infoStylesDefault,
          success: infoStylesSuccess,
          warning: infoStylesWarning,
        },
      };

      return styles;
    },
    // sortedItems: function () {
    //   function compare(a, b) {
    //     if (a.order < b.order) return -1;
    //     if (a.order > b.order) return 1;
    //     return 0;
    //   }
    //   let itemsCopy = this.items;
    //   return itemsCopy.sort(compare);
    // },
  },
  methods: {
    handleElementsChanged(data) {
      this.items = data.newElementsArray;
    },
    handleElementClick() {
      // Do stuff when user clicks on element without dragging it
      // ...
    },
  },
};
</script>

<style lang="scss">
$gray: #aaa;
$light-gray: #fafafa;
.controls {
  margin-bottom: 16px;
  padding: 16px;

  .button {
    display: inline-block;
    margin: 4px 2px;
    border: none;
    border-radius: 8px;
    padding: 15px 32px;
    text-align: center;
    font-size: 16px;
    text-decoration: none;
    color: white;
    background-color: #555555;
    cursor: pointer;
    transition: all 0.1s ease-in;

    &.active {
      background-color: #4caf50;
    }
  }
}

.lists-container {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
  gap: 32px;
  padding: 64px;
  border: 2px dashed gray;
  border-radius: 8px;

  .drag-container-vertical {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    padding: 16px;
    border: 2px solid gray;
    border-radius: 8px;

    .drag-element {
      display: flex;
      align-items: center;
      width: 200px;
      border: 1px solid $gray;
      border-radius: 8px;
      padding: 32px;
      transition: opacity 0.05s ease-in;
      background-color: $light-gray;
      user-select: none;
      opacity: 1;

      // &.element-is-dragging {
      //   // Customize drag element clone styling...
      // }

      .handle {
        background-color: hotpink;
        margin-right: 8px;
      }

      &:hover {
        cursor: grab;
      }

      &.pre-drag {
        opacity: 0.8;
        // Customize drag element pre-drag styling...
      }

      &:not(:last-child) {
        margin-bottom: 16px;
      }
    }

    // .element-drop-preview {
    //   // Customize drop preview styling...
    // }
  }

  .drag-container-horizontal {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    height: 200px;
    padding: 16px;
    border: 2px solid gray;
    border-radius: 8px;

    .drag-element {
      display: flex;
      align-items: center;
      width: 200px;
      border: 1px solid $gray;
      border-radius: 8px;
      padding: 32px;
      transition: opacity 0.05s ease-in;
      background-color: $light-gray;
      user-select: none;
      opacity: 1;

      // &.element-is-dragging {
      //   // Customize drag element clone styling...
      // }

      .handle {
        background-color: hotpink;
        margin-right: 8px;
      }

      &:hover {
        cursor: grab;
      }

      &.pre-drag {
        opacity: 0.8;
        // Customize drag element pre-drag styling...
      }

      &:not(:last-child) {
        margin-right: 16px;
      }
    }

    // .element-drop-preview {
    //   // Customize drop preview styling...
    // }
  }
}
</style>
