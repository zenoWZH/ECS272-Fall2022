<template>
     <a-layout-header class="header">
     
      <a-menu
        :selectedKeys="selectedKey"
        theme="dark"
        mode="horizontal"
        :style="{ lineHeight: '64px' }"
      >
        <a-menu-item key="1"> 
             <router-link to="/">Home</router-link>
        </a-menu-item>
        <a-menu-item key="2">
             <router-link to="/about">About</router-link>
        </a-menu-item>
        <a-menu-item key="3">
             <router-link to="/third">third</router-link>
        </a-menu-item>
        <a-menu-item key="4">
             <router-link to="/trend">Trend</router-link>
        </a-menu-item>
        <a-menu-item key="4">
             <router-link to="/explore">Explore</router-link>
        </a-menu-item>
      </a-menu>
    </a-layout-header>
</template>

<script setup lang="ts">
    import {ref} from 'vue';
    var selectedKey = ref(['1'])


</script>

<style scoped>

</style>