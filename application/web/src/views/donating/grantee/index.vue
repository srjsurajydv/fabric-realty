<template>
  <div class="container">
    <el-alert
      type="success"
    >
      <p>Account_ID: {{ accountId }}</p>
      <p>Username: {{ userName }}</p>
      <p>Balance: INR{{ balance }}</p>
    </el-alert>
    <div v-if="donatingList.length==0" style="text-align: center;">
      <el-alert
        title="Can't find data"
        type="warning"
      />
    </div>
    <el-row v-loading="loading" :gutter="20">
      <el-col v-for="(val,index) in donatingList" :key="index" :span="6" :offset="1">
        <el-card class="d-buy-card">
          <div slot="header" class="clearfix">
            <span>{{ val.donating.donatingStatus }}</span>
            <el-button v-if="val.donating.donatingStatus==='Donating'" style="float: right; padding: 3px 0" type="text" @click="updateDonating(val,'done')">Confirm</el-button>
            <el-button v-if="val.donating.donatingStatus==='Donating'" style="float: right; padding: 3px 6px" type="text" @click="updateDonating(val,'cancelled')">Cancel </el-button>
          </div>
         <div align="left">
          <div class="item">
            <el-tag>Property_ID: </el-tag>
            <span>  {{ val.donating.objectOfDonating }}</span>
          </div>
          <div class="item">
            <el-tag type="success">Donor_ID: </el-tag>
            <span>  {{ val.donating.donor }}</span>
          </div>
          <div class="item">
            <el-tag type="danger">Grantee_ID: </el-tag>
            <span>  {{ val.donating.grantee }}</span>
          </div>
          <div class="item">
            <el-tag type="warning">Creation Time: </el-tag>
            <span>  {{ val.donating.createTime }}</span>
          </div>
        </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import { queryDonatingListByGrantee, updateDonating } from '@/api/donating'

export default {
  name: 'DonatingGrantee',
  data() {
    return {
      loading: true,
      donatingList: []
    }
  },
  computed: {
    ...mapGetters([
      'accountId',
      'userName',
      'balance'
    ])
  },
  created() {
    queryDonatingListByGrantee({ grantee: this.accountId }).then(response => {
      if (response !== null) {
        this.donatingList = response
      }
      this.loading = false
    }).catch(_ => {
      this.loading = false
    })
  },
  methods: {
    updateDonating(item, type) {
      let tip = ''
      if (type === 'done') {
        tip = 'confirm acceptance of donation'
      } else {
        tip = 'Cancel donation operation'
      }
      this.$confirm('Do you want to ' + tip + '?', 'hint', {
        confirmButtonText: 'Sure',
        cancelButtonText: 'Cancel',
        type: 'success'
      }).then(() => {
        this.loading = true
        updateDonating({
          donor: item.donating.donor,
          grantee: item.donating.grantee,
          objectOfDonating: item.donating.objectOfDonating,
          status: type
        }).then(response => {
          this.loading = false
          if (response !== null) {
            this.$message({
              type: 'success',
              message: tip + 'Successful Operation!'
            })
          } else {
            this.$message({
              type: 'error',
              message: tip + 'Operation Failed!'
            })
          }
          setTimeout(() => {
            window.location.reload()
          }, 1000)
        }).catch(_ => {
          this.loading = false
        })
      }).catch(() => {
        this.loading = false
        this.$message({
          type: 'info',
          message: 'Cancelled' + tip
        })
      })
    }
  }
}

</script>

<style>
  .container{
    width: 100%;
    text-align: center;
    min-height: 100%;
    overflow: hidden;
  }
  .tag {
    float: left;
  }

  .item {
    font-size: 14px;
    margin-bottom: 18px;
    color: #999;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
  }
  .clearfix:after {
    clear: both
  }

  .d-buy-card {
    width: 280px;
    height: 300px;
    margin: 18px;
  }
</style>
