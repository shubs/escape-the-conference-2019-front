<template>
  <div class="home-page">
    <div class="sidebar">
      <div class="brand-logo-container mb-2 mt-2">
        <img src="@/assets/brand-logo.png" alt="">
      </div>
      <div class="total-players-container flex flex-col items-center w-full mt-2">
        <div class="label bg-grey text-white w-full text-center text-3xl">
          TOTAL PLAYERS
        </div>
        <div class="count text-2xl">
          {{users.length}}
        </div>
      </div>
      <div class="total-validation-attempts-container flex flex-col items-center w-full  mt-2">
        <div class="label bg-grey text-white w-full text-center text-3xl">
          TOTAL VALIDATION ATTEMPTS
        </div>
        <div class="count text-2xl">
          {{totalValidationAttempts}}
        </div>
      </div>
      <div class="top-bruteforcers-container flex flex-col items-center w-full  mt-2">
        <div class="label bg-grey text-white w-full text-center text-3xl">
          TOP BRUTEFORCERS
        </div>
        <div class="bruteforcers-container flex flex-col w-full mt-4">
            <template v-for="user in topBruteForcers">
              <div class="bruteforcer-container flex flex-row items-center" :key="'bubble' + user._id">
                <UserBubble :user="user" :is-leader="true" class="mb-2"></UserBubble>
                <span class="flex-1"></span>
                <div class="user-attempts text-2xl mr-2">{{user.validationAttempts}}</div>
              </div>
            </template>
          </div>
      </div>
    </div>
    <div class="levels-chart-container">
      <div class="levels-rows-container">
        <div class="level-row flex-1 relative items-center" v-for="index in levelsCount + 1" :key="index">
          <div class="level-info-container flex flex-col bg-red text-3xl bg-grey text-white self-stretch text-center justify-center">
            <span>{{index-1}}</span>
          </div>
          <transition-group name="list-complete" class="ml-2">
            <UserBubble  :is-leader="isLeader(user._id)" class="user-bubble list-complete-item" v-for="user in getUsers(index -1)" :key="user._id" :user="user">
              <!-- <span>{{minTimesEllapsed[user._id]}}</span> -->
            </UserBubble>
          </transition-group>
        </div>
          <div class="url-label text-5xl w-full flex-row-reverse flex flex-1 pr-4">
           <span>http://escape.apidays.io/maze</span>
           <span class="flex-1"></span>
            <span class="url-spacer self-stretch bg-grey"></span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import uuid from 'uuid/v1'
import UserBubble from '@/components/UserBubble'
import { random, clone, cloneDeep } from 'lodash'

import apiConfig from '@/config/api'

function getLevel (levels) {
  return levels.filter(level => level === true).length
}

function makeRandomUser () {
  return {
    id: uuid(),
    email: uuid() + '@' + uuid() + '.fr',
    'levels': [
      false,
      false,
      false,
      false,
      false,
      false,
      false
    ],
    validationTimestamps: [0, 0, 0, 0, 0, 0, 0],
    creationTimestamp: Date.now(),
    'validationAttempts': 0
  }
}
export default {
  name: 'home',
  components: {
    UserBubble
  },
  mounted () {
    const interval = this.$route.query.interval ? parseInt(this.$route.query.interval) : 1000
    window.setInterval(() => {
      this.axios.get(`${apiConfig.endpoint}/users`)
        .then(res => {
          this.users = res.data
          // console.log('USERS RECEIVED ' + res.data.length)
        })
    }, interval)
    // window.setInterval(() => {
    //   this.addUser()
    //   this.simulateValidation()
    // }, 100)
  },
  data () {
    return {
      levelsCount: 7,
      gridCols: 50,
      users: [],
      minTimesEllapsed: {},
      leaderUsersIds: []
    //   users: [
    //     {
    //       'id': 'fVQG4mGT5uoffdhipNu',
    //       'email': 'shubham@sharma.fr',
    //       'levels': [
    //         false,
    //         false,
    //         false,
    //         false,
    //         false,
    //         false,
    //         false
    //       ],
    //       'validationAttempts': 10
    //     },
    //     {
    //       'id': 'fdsmoo',
    //       'email': 'shubham@shar1ma.fr',
    //       'levels': [
    //         false,
    //         false,
    //         true,
    //         false,
    //         false,
    //         false,
    //         false
    //       ],
    //       'validationAttempts': 10
    //     },
    //     {
    //       'id': 'defdorgnoahroeoa',
    //       'email': 'shubham@shar1ma.fr',
    //       'levels': [
    //         false,
    //         true,
    //         true,
    //         false,
    //         false,
    //         false,
    //         false
    //       ],
    //       'validationAttempts': 10
    //     },
    //     {
    //       'id': 'fdsegdsgt',
    //       'email': 'shubham@shar1ma.fr',
    //       'levels': [
    //         false,
    //         true,
    //         true,
    //         false,
    //         false,
    //         false,
    //         false
    //       ],
    //       'validationAttempts': 10
    //     },
    //     {
    //       'id': 'desfes',
    //       'email': 'shubham@shar1ma.fr',
    //       'levels': [
    //         false,
    //         true,
    //         true,
    //         false,
    //         false,
    //         false,
    //         false
    //       ],
    //       'validationAttempts': 10
    //     }
    //   ],
    //   disableCellTransition: false
    }
  },
  computed: {
    processedUsers () {
      return this.users.map(user => {
        // console.log(getLevel(user.levels))
        const processedUser = {
          level: getLevel(user.levels),
          ...user
        }
        return processedUser
      })
    },
    totalValidationAttempts () {
      return this.users.map(u => u.validationAttempts).reduce((prev, curr) => prev + curr, 0)
    },
    topBruteForcers () {
      const users = cloneDeep(this.users)
      const sortedUsers = users.sort((a, b) => b.validationAttempts - a.validationAttempts)
      const topUsers = new Array(5).fill({}).map((val, i) => sortedUsers[i])
      return topUsers
    },
    leaderUsersId () {
      const leaderboard = new Array(this.levelsCount).fill(0).map((val, i, array) => {
        return this.getUsers(array.length - i)
      }).flat()
      // console.log('LEADERBOARD')
      // console.log(leaderboard)
      return leaderboard
    }
  },
  watch: {
    processedUsers (newVal, oldVal) {
      newVal.map(u => {
        const min = u.validationTimestamps.filter(v => v !== 0).sort((a, b) => a - b).reduce((acc, curr, index, array) => {
          const nextTimestamp = array[index + 1] ? array[index + 1] : 0
          const delta = acc + curr - nextTimestamp
          return delta
        }, 0)
        this.minTimesEllapsed[u._id] = min
        return min
      })

      const leaderboard = new Array(this.levelsCount).fill(0).map((val, i, array) => {
        return this.getUsers(array.length - i)
      }).flat()
      // console.log('LEADERBOARD')
      this.leaderUsersIds = leaderboard.map(u => u._id)
      // console.log(this.leaderUsersId)

      // console.log(minTimesEllapsed)
    }
  },
  methods: {
    getUsers (level) {
      // console.log(level + '===>')
      // console.log(this.users)
      let levelUsers = this.processedUsers.filter(user => user.level === level)
      if (level === 0) {
        levelUsers = levelUsers.sort((a, b) => {
          return a.creationTimestamp - b.creationTimestamp
        })
      } else {
        levelUsers = levelUsers.sort((a, b) => {
          return a.totalLevelsTimestamps[level] - b.totalLevelsTimestamps[level]
        })
      }

      return levelUsers
    },
    simulateValidation () {
      const randUserIndex = random(0, this.users.length - 1)
      const randLevelIndex = random(0, this.levelsCount - 1)
      const randUser = clone(this.users[randUserIndex])
      if (randUser.levels[randLevelIndex] === false) randUser.validationTimestamps[randLevelIndex] = Date.now()
      randUser.levels[randLevelIndex] = true
      this.$set(this.users, randUserIndex, randUser)
    },
    addUser () {
      this.users.push(makeRandomUser())
    },
    getCellRow (level) {
      const users1 = this.getUsers(level).map(user => {
        return {
          user
        }
      })
      const reminder1 = this.users.length - users1.length
      const reminderCells1 = new Array(reminder1).fill(null).map((val, index) => {
        return {
          user: {
            email: 'fake' + uuid()
          }
        }
      })
      const row1 = users1.concat(reminderCells1)
      return row1
    },
    isLeader (id) {
      return this.leaderUsersId[0]._id === id || this.leaderUsersId[1]._id === id || this.leaderUsersId[2]._id === id
    },
    getLeaderPos (id) {
      return this.leaderUsersId.findIndex(user => user._id === id)
    }
  }
}
</script>

<style lang="postcss" scoped>

.home-page {
  width: 100vw;
  min-height: 100vh;
  display: flex;
  flex-direction: row;
}

.sidebar {
  width: 500px;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-right: 4px solid #111;
}

.levels-chart-container {
  width: 100%;
  min-height: 100vh;
}

.user-container {
  width: 100%;
  height: 50%;
  background: green;
}
.levels-rows-container {
  display: flex;
  flex-direction: column-reverse;
  width: 100%;
  height: 100%;
  min-height: 100vh;
}

.level-row {
  display: flex;
  flex-direction: row;
  width: 100%;
}

.list-complete-item {
  transition: all 0.3s ease-in-out;
  display: inline-block;
  /* margin-right: 10px; */
}
.list-complete-enter
/* .list-complete-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(calc(100vh / 14)) scale(0.7);
}

.list-complete-leave-to {
  opacity: 0;
  transform: translateY(calc(-100vh / 14)) scale(0.7);
}
.list-complete-leave-active {
  position: absolute;
}

.level-info-container {
  font-weight: bold;
  min-width: 50px;
  border-right: 4px solid #111;
}

.brand-logo-container {
  width: 200px;
}

.url-spacer {
  width: 50px;
}

</style>
