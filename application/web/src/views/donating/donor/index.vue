<template>
  <div class="container">
    <el-alert
      type="success"
    >
      <p>Account_ID: {{ accountId }}</p>
      <p>Username: {{ userName }}</p>
      <p>Balance: INR {{ balance }}</p>
    </el-alert>
    <div v-if="donatingList.length==0" style="text-align: center;">
      <el-alert
        title="Can't find data"
        type="warning"
      />
    </div>
    <el-row v-loading="loading" :gutter="20">
      <el-col v-for="(val,index) in donatingList" :key="index" :span="6" :offset="1">
        <el-card class="d-me-card">
          <div slot="header" class="clearfix">
            <span>{{ val.donatingStatus }}</span>
            <el-button v-if="val.donatingStatus === 'Donating'" style="float: right; padding: 3px 0" type="text" @click="updateDonating(val)">Cancel</el-button>
          </div>
         <div align="left">
          <div class="item">
            <el-tag>Property_ID: </el-tag>
            <span>  {{ val.objectOfDonating }}</span>
          </div>
          <div class="item">
            <el-tag type="success">Donor_ID: </el-tag>
            <span>  {{ val.donor }}</span>
          </div>
          <div class="item">
            <el-tag type="danger">Grantee_ID: </el-tag>
            <span>  {{ val.grantee }}</span>
          </div>
          <div class="item">
            <el-tag type="warning">Creation Time: </el-tag>
            <span>  {{ val.createTime }}</span>
          </div>
        </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import { queryDonatingList, updateDonating } from '@/api/donating'

export default {
  name: 'DonatingDonor',
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
    queryDonatingList({ donor: this.accountId }).then(response => {
      if (response !== null) {
        this.donatingList = response
      }
      this.loading = false
    }).catch(_ => {
      this.loading = false
    })
  },
  methods: {
    updateDonating(item) {
      this.$confirm('Do you want to cancel the donation?', 'hint', {
        confirmButtonText: 'Sure',
        cancelButtonText: 'Cancel',
        type: 'success'
      }).then(() => {
        this.loading = true
        updateDonating({
          donor: item.donor,
          grantee: item.grantee,
          objectOfDonating: item.objectOfDonating,
          status: 'cancelled'
        }).then(response => {
          this.loading = false
          if (response !== null) {
            this.$message({
              type: 'success',
              message: 'Successful operation!'
            })
          } else {
            this.$message({
              type: 'error',
              message: 'Operation failed!'
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
          message: 'Operation Canceled!'
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

  .d-me-card {
    width: 280px;
    height: 300px;
    margin: 18px;
  }
</style>
