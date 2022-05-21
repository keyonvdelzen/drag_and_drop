<template>
  <div class="lists-container">
    <button @click="testLog()">test</button>
    <drag-component
      :elements="items"
      @elements-changed="updateItems"
      :dragDelay="50"
      dragContainerId="drag-container"
      dragItemClass="drag-item"
      preDragItemClass="pre-drag"
    >
      <div class="drag-container" id="drag-container">
        <div v-for="item in items" :key="item.id" class="drag-item">
          <div class="content">
            <div class="text">{{ item.text }}</div>
            <div class="footer">
              <div>
                part 1
                <div style="text-decoration: underline">sadf</div>
                <div>asdf</div>
              </div>
              <div>part 2</div>
            </div>
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
          text: "AAAAAA AAAAAAAAAAA AAAAAAA AAAAAAAAA",
          order: 0,
          clone: false,
        },
        {
          id: 1,
          text: "BBBBBB BBBBBBBBBB BBBBBBBBBBB BBBBBBB",
          order: 1,
          clone: false,
        },
        {
          id: 2,
          text: "CCCCCCC CCCCCCCCCCCCC CCCCCCCCC CCCCC",
          order: 2,
          clone: false,
        },
        {
          id: 3,
          text: "DDDDDD DDDDDDD DDDDD DDDDD DDDDDDD DDDDDDD",
          order: 3,
          clone: false,
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
    testLog() {
      console.table(this.items, "text");
    },
    updateItems(newValue) {
      // console.table(newValue, "text");
      this.items = newValue;
      // console.table(this.items, "text");
    },
  },
};
</script>

<style lang="scss" scoped>
$gray: #aaa;
$light-gray: #fafafa;

.lists-container {
  display: flex;
  justify-content: center;
  gap: 32px;

  .drag-container {
    display: flex;
    flex-direction: column;
    max-width: 350px;
    padding: 16px;
    border: 1px solid black;

    .drag-item {
      display: flex;
      align-items: center;
      border: 1px dashed $gray;
      padding: 16px;
      transition: opacity 0.1s ease-in;
      background-color: $light-gray;
      user-select: none;

      .handle {
        background-color: hotpink;
        margin-right: 8px;
      }

      .footer {
        :first-child {
          background-color: lightcoral;

          :first-child {
            background-color: purple;
          }
        }
        :last-child {
          background-color: lightcyan;
        }
      }

      &:hover {
        cursor: grab;
      }

      &.pre-drag {
        opacity: 0.8;
      }

      &:not(:last-child) {
        margin-bottom: 16px;
      }
    }
  }
}
</style>
