<template>
  <div id="app">
    <div v-click-outside="onClickOutside">
      <Editor
        @valueChanged="onValueChanged($event)"
        :selected="selectedElement"
      />
      <TextContent
        :data="data"
        @mouseDown="onMouseDown($event)"
        @mouseUp="onTextSelected($event)"
      />
    </div>

    <json-viewer :value="data" :expand-depth="5"></json-viewer>
  </div>
</template>

<script>
import vClickOutside from "v-click-outside";

import Editor from "./components/editor";
import TextContent from "./components/textContent";

export default {
  name: "App",
  components: {
    Editor,
    TextContent,
  },
  mounted() {
    window.addEventListener("keyup", this.handleKeyUp);
  },

  data() {
    return {
      data: [
        {
          text: "My lovely little Ponny",
          fontSize: "45px",
          color: "black",
          bcgColor: "pink",
        },
      ],
      selectedText: "",
      selectedElement: null,
      cursorOffset: null,
      start: null,
      end: null,
    };
  },
  directives: {
    clickOutside: vClickOutside.directive,
  },
  methods: {
    onMouseDown(e) {
      setTimeout(() => {
        this.start = e;
        this.cursorOffset = window.getSelection().anchorOffset;
      });
    },
    handleKeyUp(e) {
      e.preventDefault();

      if (this.start == null) {
        return;
      }

      const keycode = e.keyCode;

      if (
        (keycode > 47 && keycode < 58) || // number keys
        keycode == 32 ||
        keycode == 13 || // spacebar & return key(s) (if you want to allow carriage returns)
        (keycode > 64 && keycode < 91) || // letter keys
        (keycode > 95 && keycode < 112) || // numpad keys
        (keycode > 185 && keycode < 193) || // ;=,-./` (in order)
        (keycode > 218 && keycode < 223)
      ) {
        this.data[this.start].text =
          this.data[this.start].text.slice(0, this.cursorOffset) +
          e.key +
          this.data[this.start].text.slice(this.cursorOffset);
        this.cursorOffset++;
      } else if (keycode == 8) {
        if (this.cursorOffset == 0 && this.start == 0) {
          return;
        }
        if (this.cursorOffset == 0) {
          this.start--;
          this.cursorOffset = this.data[this.start].text.length;
        }

        this.data[this.start].text =
          this.data[this.start].text.slice(0, this.cursorOffset - 1) +
          this.data[this.start].text.slice(this.cursorOffset);
        this.cursorOffset--;
      } else if (keycode == 46) {
        if (
          this.start == this.data.length - 1 &&
          this.cursorOffset == this.data[this.start].text.length
        ) {
          return;
        }
        if (this.cursorOffset == this.data[this.start].text.length) {
          this.start++;
          this.cursorOffset = 0;
        }

        this.data[this.start].text =
          this.data[this.start].text.slice(0, this.cursorOffset) +
          this.data[this.start].text.slice(this.cursorOffset + 1);
      }
      this.regroupData();
    },
    onValueChanged(newVal) {
      if (!this.selectedElement) {
        return;
      }
      let exestingElementIndex = this.data.findIndex(
        (item) => item.text == this.selectedText
      );
      if (exestingElementIndex != -1) {
        this.data[exestingElementIndex][newVal.type] = newVal.value;
      } else {
        this.updateData(newVal);
      }
      this.regroupData();
    },
    onTextSelected(end) {
      if (!window.getSelection()) {
        return;
      }

      this.selectedElement = null;
      this.selectedText = window.getSelection().toString();
      this.end = end;

      if (this.start > this.end) {
        let temp = this.end;
        this.end = this.start;
        this.start = temp;
      }
      this.checkTextParams();
    },
    checkTextParams() {
      this.selectedElement = this.data[this.start];
    },
    updateData(newVal) {
      if (
        this.start == this.end &&
        this.selectedText.length < this.data[this.start].text.length
      ) {
        this.updateDataSingleElement(newVal);
      } else {
        this.updateMultipleElements(newVal);
      }
      this.cancelSelection();
    },
    updateDataSingleElement(newVal) {
      let indexOfSelected = this.data[this.start].text.indexOf(
        this.selectedText
      );
      const newItem = {
        ...this.data[this.start],
        text: this.selectedText,
        [newVal.type]: newVal.value,
      };
      if (
        indexOfSelected + this.selectedText.length ==
        this.data[this.start].text.length
      ) {
        this.data[this.start].text = this.data[this.start].text.replace(
          this.selectedText,
          ""
        );
        this.data = [
          ...this.data.slice(0, this.start + 1),
          newItem,
          ...this.data.slice(this.start + 1),
        ];
      } else if (indexOfSelected == 0) {
        this.data[this.start].text = this.data[this.start].text.replace(
          this.selectedText,
          ""
        );

        this.data = [
          ...this.data.slice(0, this.start),
          newItem,
          this.data[this.start],
          ...this.data.slice(this.start + 1),
        ];
      } else {
        const elementBefore = { ...this.data[this.start] };
        elementBefore.text = elementBefore.text.substring(0, indexOfSelected);

        const elementAfter = { ...this.data[this.start] };
        elementAfter.text = elementAfter.text.substring(
          indexOfSelected + newItem.text.length
        );
        this.data = [
          ...this.data.slice(0, this.start),
          elementBefore,
          newItem,
          elementAfter,
          ...this.data.slice(this.start + 1),
        ];
      }
    },
    updateMultipleElements(newVal) {
      const newItem = {
        ...this.data[this.start],
        text: this.selectedText,
        [newVal.type]: newVal.value,
      };
      if (this.end - this.start > 1) {
        let removedItems = this.data.slice(this.start + 1, this.end);
        let removedItemsText = "";
        removedItems.forEach((t) => {
          removedItemsText += t.text;
        });

        const textForClear = this.selectedText.split(removedItemsText);

        const elementBefore = { ...this.data[this.start] };
        elementBefore.text = elementBefore.text.replace(textForClear[0], "");

        const elementAfter = { ...this.data[this.end] };
        elementAfter.text = elementAfter.text.replace(textForClear[1], "");

        this.data = [
          ...this.data.slice(0, this.start),
          elementBefore,
          newItem,
          elementAfter,
          ...this.data.slice(this.end + 1),
        ];
      } else {
        const textStartsAt = this.findSelectedFragmentIndex(
          this.data[this.start].text,
          this.selectedText.slice(0, -1)
        );
        if (textStartsAt == -1) {
          this.cancelSelection();
          return;
        }

        const elementBefore = { ...this.data[this.start] };
        elementBefore.text = elementBefore.text.slice(0, textStartsAt);

        const textRemoveAfterElement =
          this.data[this.start].text.length - elementBefore.text.length;

        const elementAfter = { ...this.data[this.end] };
        elementAfter.text = elementAfter.text.slice(textRemoveAfterElement);

        this.data = [
          ...this.data.slice(0, this.start),
          elementBefore,
          newItem,
          elementAfter,
          ...this.data.slice(this.end + 1),
        ];
      }
    },
    regroupData() {
      for (let i = 0; i < this.data.length; i++) {
        let curr = this.data[i];
        let next = this.data[i + 1];

        if (
          next &&
          curr.color == next.color &&
          curr.fontSize == next.fontSize &&
          curr.bcgColor == next.bcgColor
        ) {
          this.data[i].text += next.text;
          this.data.splice(i + 1, 1);
          i--;
        }
      }
      this.data = this.data.filter((item) => item.text !== "");
    },
    cancelSelection() {
      this.selectedText = null;
      this.selectedElement = null;
      this.start = null;
      this.end = null;
      this.cursorOffset = null;
      if (window.getSelection().empty) {
        // Chrome
        window.getSelection().empty();
      } else if (window.getSelection().removeAllRanges) {
        // Firefox
        window.getSelection().removeAllRanges();
      }
    },
    findSelectedFragmentIndex(fragment, query) {
      if (query) {
        const match = fragment.match(query);
        if (match) {
          return match.index;
        } else {
          return this.findSelectedFragmentIndex(fragment, query.slice(0, -1));
        }
      } else {
        return -1;
      }
    },
    onClickOutside() {
      this.cancelSelection();
    },
  },
};
</script>

<style lang="scss">
.jv-container.jv-light {
  background-color: #eee;
}
</style>
