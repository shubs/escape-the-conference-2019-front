<template>
    <div class="user-bubble-component" :class="{'is-leader': isLeader}">
      <div class="user-row-container">
        <div class="avatar-bubble-container">
          <v-gravatar class="avatar-bubble rounded-full rounded-full shadow-xl mx-1" :email="user.email" :style="userStyleObject"></v-gravatar>
        </div>
        <div class="user-info-container p-3 shadow-xl rounded-full bg-grey" v-if="isLeader">{{user.email}}</div>

          <slot></slot>
      </div>
      </div>
</template>

<script>
import ColorHash from 'color-hash'

export default {
  props: {
    user: Object,
    isLeader: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    userStyleObject () {
      return {
        borderColor: new ColorHash().hex(this.user._id)
      }
    }
  }
}
</script>

<style lang="postcss" scoped>
.user-row-container {
   display: flex;
   flex-direction: row;
   align-items: center;
}

.avatar-bubble {
  width: 50px;
  height: 50px;
  z-index: 5;
  position: relative;
  border: 5px solid;
}

.user-info-container {
  margin-left: -50px;
  padding-left: 60px;
  padding-right: 20px;
  color: white;
  font-weight: bold;
}

.is-leader {
}
</style>
