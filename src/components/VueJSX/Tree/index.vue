<script lang="tsx">
import { defineComponent, reactive } from "vue";
import TreeItem from "./TreeItem.vue";
export default defineComponent({
  name: "JSXTree",
  components: { TreeItem },
  setup() {
    const treeData = reactive({
      name: 'My Tree JSX',
      children: [
        { name: 'hello' },
        { name: 'wat' },
        {
          name: 'child folder',
          children: [
            {
              name: 'child folder',
              children: [
                { name: 'hello' },
                { name: 'wat' }
              ]
            },
            { name: 'hello' },
            { name: 'wat' },
            {
              name: 'child folder',
              children: [
                { name: 'hello' },
                { name: 'wat' }
              ]
            }
          ]
        }
      ]
    });
    return () => (
      <ul>
        <TreeItem class="item" model={treeData}></TreeItem>
      </ul>
    );
  },
});
</script>
<style>
  .item {
    cursor: pointer;
  }
  .bold {
    font-weight: bold;
  }
  ul {
    padding-left: 1em;
    line-height: 1.5em;
    list-style-type: dot;
  }
</style>
