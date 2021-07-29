<template>
  <div>
    <SwipeItem v-for="item in 100" :key="item" :data-key="item">
      {{ item }}
    </SwipeItem>
  </div>
</template>
