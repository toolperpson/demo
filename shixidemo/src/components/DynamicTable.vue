<template>
    <div>
      <!-- 搜索和筛选 -->
      <div style="margin-bottom: 16px;">
        <el-input v-model="searchId" placeholder="搜索ID" style="width: 200px; margin-right: 8px;" @input="handleSearch"/>
        <el-select v-model="filterColumn" placeholder="筛选列" style="width: 120px; margin-right: 8px;">
          <el-option label="baseMean" value="baseMean"/>
          <el-option label="log2FoldChang" value="log2FoldChang"/>
          <el-option label="lfcSE" value="lfcSE"/>
          <el-option label="stat" value="stat"/>
          <el-option label="pvalue" value="pvalue"/>
          <el-option label="padj" value="padj"/>
        </el-select>
        <el-select v-model="filterType" placeholder="筛选类型" style="width: 120px; margin-right: 8px;">
          <el-option label="大于" value="gt"/>
          <el-option label="小于" value="lt"/>
        </el-select>
        <el-input-number v-model="filterValue" placeholder="数值" style="width: 120px; margin-right: 8px;"/>
        <el-button @click="handleFilter">筛选</el-button>
        <el-button @click="resetFilter" type="info">重置</el-button>
      </div>
      <!-- 表格 -->
      <el-table :data="pagedData" @sort-change="handleSort" style="width: 100%;">
        <el-table-column prop="ID" label="ID" sortable />
        <el-table-column prop="baseMean" label="baseMean" sortable />
        <el-table-column prop="log2FoldChang" label="log2FoldChang" sortable />
        <el-table-column prop="lfcSE" label="lfcSE" sortable />
        <el-table-column prop="stat" label="stat" sortable />
        <el-table-column prop="pvalue" label="pvalue" sortable />
        <el-table-column prop="padj" label="padj" sortable />
      </el-table>
      <!-- 分页 -->
      <el-pagination
        :page-size="pageSize"
        :current-page="currentPage"
        :total="filteredData.length"
        @current-change="handlePageChange"
        @size-change="handleSizeChange"
        :page-sizes="[10, 20, 50]"
        layout="total, sizes, prev, pager, next, jumper"
        style="margin-top: 16px;"
      />
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted } from 'vue';
  import { readExcel } from '../utils/excel';
  
  const tableData = ref([]);
  const filteredData = ref([]);
  const pagedData = ref([]);
  const pageSize = ref(10);
  const currentPage = ref(1);
  
  const searchId = ref('');
  const filterType = ref('');
  const filterValue = ref(null);
  const filterColumn = ref('baseMean'); // 默认筛选baseMean
  
  onMounted(async () => {
    tableData.value = await readExcel('/front-end-dynamic-table.xlsx');
    filteredData.value = tableData.value;
    updatePagedData();
  });
  
  function handleSort({ prop, order }) {
    if (!order) return;
    filteredData.value.sort((a, b) => {
      if (order === 'ascending') return a[prop] > b[prop] ? 1 : -1;
      else return a[prop] < b[prop] ? 1 : -1;
    });
    updatePagedData();
  }
  
  function handlePageChange(page) {
    currentPage.value = page;
    updatePagedData();
  }
  
  function handleSizeChange(size) {
    pageSize.value = size;
    currentPage.value = 1;
    updatePagedData();
  }
  
  function handleSearch() {
    if (searchId.value) {
      filteredData.value = tableData.value.filter(row => String(row.ID).includes(searchId.value));
    } else {
      filteredData.value = [...tableData.value];
    }
    currentPage.value = 1;
    updatePagedData();
  }
  
  function handleFilter() {
    if (filterType.value && filterValue.value !== null && filterColumn.value) {
      filteredData.value = tableData.value.filter(row => {
        const val = parseFloat(row[filterColumn.value]);
        if (isNaN(val)) return false;
        if (filterType.value === 'gt') return val > filterValue.value;
        if (filterType.value === 'lt') return val < filterValue.value;
        return true;
      });
    }
    currentPage.value = 1;
    updatePagedData();
  }
  
  function resetFilter() {
    searchId.value = '';
    filterType.value = '';
    filterValue.value = null;
    filterColumn.value = 'baseMean'; // 重置筛选列
    filteredData.value = [...tableData.value];
    currentPage.value = 1;
    updatePagedData();
  }
  
  function updatePagedData() {
    const start = (currentPage.value - 1) * pageSize.value;
    pagedData.value = filteredData.value.slice(start, start + pageSize.value);
  }
  </script>