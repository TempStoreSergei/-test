<template>
  <div class="wrapper">
    <div class="container">
      <h1>Города Российской Федерации</h1>
      <RadioGroup v-model:value="searchType" class="radio-group">
        <Radio value="city">По названию города</Radio>
        <Radio value="region">По названию региона</Radio>
      </RadioGroup>
      <InputSearch v-model:value="searchValue" placeholder="Поиск" class="search-input" />
      <Tree
          :expanded-keys="expandedKeys"
          :auto-expand-parent="autoExpandParent"
          :tree-data="filteredData"
          @expand="onExpand"
          :height="400"
          class="custom-tree"
      >
        <template #switcherIcon="{ switcherCls }">
          <DownOutlined :class="switcherCls" />
        </template>
        <template #title="{ title }">
          <span v-html="highlightSearch(title)" class="tree-title"></span>
        </template>
      </Tree>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { Radio, InputSearch, Tree, RadioGroup } from "ant-design-vue";
import { DownOutlined } from "@ant-design/icons-vue";
import { data } from './../mocks/data';

const searchType = ref('city');
const searchValue = ref('');
const expandedKeys = ref([]);
const autoExpandParent = ref(true);
const gData = ref(data.sort((a, b) => a.title.localeCompare(b.title)));

const highlightSearch = (title) => {
  if (!searchValue.value) return title;
  const index = title.toLowerCase().indexOf(searchValue.value.toLowerCase());
  if (index === -1) return title;
  const start = title.substring(0, index);
  const matched = title.substring(index, index + searchValue.value.length);
  const end = title.substring(index + searchValue.value.length);
  return `${start}<span class="highlight">${matched}</span>${end}`;
};

const onExpand = (keys) => {
  expandedKeys.value = keys;
  autoExpandParent.value = true;
};

const filteredData = computed(() => {
  const filtered = {};
  gData.value.forEach(item => {
    if (searchType.value === 'city') {
      if (item.children) {
        item.children.forEach(city => {
          if (city.title.toLowerCase().includes(searchValue.value.toLowerCase())) {
            const firstLetter = city.title.charAt(0).toUpperCase();
            if (!filtered[firstLetter]) {
              filtered[firstLetter] = { title: firstLetter, key: firstLetter, children: [] };
            }
            filtered[firstLetter].children.push(city);
            if (searchValue.value !== '') {
              expandedKeys.value.push(city.key); // Add the key to expandedKeys when searchValue is not empty
            }
          }
        });
      }
    } else if (searchType.value === 'region') {
      if (item.title.toLowerCase().includes(searchValue.value.toLowerCase())) {
        filtered[item.key] = item;
        if (searchValue.value !== '') {
          expandedKeys.value.push(item.key); // Add the key to expandedKeys when searchValue is not empty
        }
      }
    }
  });
  return Object.values(filtered);
});



watch(searchType, (newValue, oldValue) => {
  searchValue.value = '';
});

</script>

<style>
.wrapper {
  font-family: "Dubai Medium", sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f0f2f5;
}


.container {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 24px;
  width: 100%;
  max-width: 600px;
}

.radio-group {
  margin-bottom: 16px;
}

.search-input {
  margin-bottom: 16px;
}

.custom-tree {
  width: 100%;
}

.tree-title {
  padding: 4px 8px;
}

.highlight {
  color: blue;
}
</style>
