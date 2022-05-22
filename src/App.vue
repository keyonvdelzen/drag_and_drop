<template>
  <div class="lists-container">
    <drag-component
      :elements="items"
      @elements-changed="handleElementsChanged"
      :dragDelay="150"
      dragContainerId="drag-container"
      dragElementClass="drag-element"
      preDragElementClass="pre-drag"
      :animateDragElement="true"
      :animateDropPreview="true"
    >
      <div class="drag-container" id="drag-container">
        <div
          v-for="item in items"
          :key="item.id"
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
      items: [
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

      // console.table(data.oldElementsArray);
      // console.table(data.newElementsArray);
    },
    handleElementClick() {
      // Do stuff when user clicks on element without dragging it
      // ...
      console.log("clicked not dragged");
    },
  },
};
</script>

<style lang="scss" scoped>
$gray: #aaa;
$light-gray: #fafafa;

.lists-container {
  display: flex;
  gap: 32px;

  .drag-container {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    padding: 16px;
    border: 1px solid black;

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

      // &.dragging {
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
