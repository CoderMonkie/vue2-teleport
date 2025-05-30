<template>
  <div :class="classes">
    <slot/>
  </div>
</template>

<script>
export default {
  name: 'teleport',
  props: {
    to: {
      type: [String, HTMLDivElement], // Accept both String and HTMLElement
      required: true,
    },
    where: {
      type: String,
      default: 'after',
    },
    disabled: Boolean,
  },
  data() {
    return {
      nodes: [],
      waiting: false,
      observer: null,
      parent: null,
      moved: false,
    };
  },
  watch: {
    to: 'maybeMove',
    where: 'maybeMove',
    disabled(value) {
      if (value) {
        this.disable();
        // Ensure all event done.
        this.$nextTick(() => {
          this.teardownObserver();
        });
      } else {
        this.bootObserver();
        this.move();
      }
    },
  },
  mounted() {
    // Store a reference to the nodes
    this.nodes = Array.from(this.$el.childNodes);

    if (!this.disabled) {
      this.bootObserver();
    }

    // Move slot content to target
    this.maybeMove();
  },
  beforeDestroy() {
    // Fix nodes reference
    this.nodes = this.getComponentChildrenNode();

    // Move back
    this.disable();

    // Stop observing
    this.teardownObserver();
  },
  computed: {
    classes() {
      if (this.disabled) {
        return ['teleporter'];
      }

      return ['teleporter', 'hidden'];
    },
  },
  methods: {
    maybeMove() {
      if (!this.disabled) {
        this.move();
      }
    },
    move() {
      this.waiting = false;

      const parent = this.to instanceof HTMLElement ? this.to : document.querySelector(this.to);

      if (!parent) {
        this.disable();

        this.waiting = true;

        return;
      }

      this.parent = parent;

      if (this.where === 'before') {
        this.parent.prepend(this.getFragment());
      } else {
        this.parent.appendChild(this.getFragment());
      }
    },
    disable() {
      if (this.parent) {
        this.$el.appendChild(this.getFragment());
        this.parent = null;
      }
    },
    // Using a fragment is faster because it'll trigger only a single reflow
    // See https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment
    getFragment() {
      const fragment = document.createDocumentFragment();

      this.nodes.forEach(node => fragment.appendChild(node));

      return fragment;
    },
    onMutations(mutations) {
      // Makes sure the move operation is only done once
      let shouldMove = false;

      for (let i = 0; i < mutations.length; i++) {
        const mutation = mutations[i];
        const filteredAddedNodes = Array.from(mutation.addedNodes).filter(node => !this.nodes.includes(node));

        if (Array.from(mutation.removedNodes).includes(this.parent)) {
          this.disable();
          this.waiting = !this.disabled;
        } else if (this.waiting && filteredAddedNodes.length > 0) {
          shouldMove = true;
        }
      }

      if (shouldMove) {
        this.move();
      }
    },
    bootObserver() {
      if (this.observer || this.to instanceof HTMLElement) {
        return; // Skip DOM observer if `to` is already an HTMLElement
      }

      this.observer = new MutationObserver(mutations => this.onMutations(mutations));

      this.observer.observe(document.body, {
        childList: true,
        subtree: true,
        attributes: false,
        characterData: false,
      });

      if (this.childObserver) {
        return;
      }
      // watch childNodes change
      this.childObserver = new MutationObserver(mutations => {
        const childChangeRecord = mutations.find(i => i.target === this.$el);
        if (childChangeRecord) {
          // Remove old nodes before update position.
          this.nodes.forEach((node) => node.parentNode && node.parentNode.removeChild(node));
          this.nodes = this.getComponentChildrenNode();
          this.maybeMove();
        }
      });

      this.childObserver.observe(this.$el, {
        childList: true,
        subtree: false,
        attributes: false,
        characterData: false,
      });
    },
    teardownObserver() {
      if (this.observer) {
        this.observer.disconnect();
        this.observer = null;
      }
      if (this.childObserver) {
        this.childObserver.disconnect();
        this.childObserver = null;
      }
    },
    getComponentChildrenNode() {
      return this.$vnode.componentOptions.children
        .map((i) => i.elm)
        .filter((i) => i);
    },
  },
};
</script>

<style scoped lang="scss">
.hidden {
  visibility: hidden;
  display: none;
}
</style>
