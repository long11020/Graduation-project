<template>
  <div id="recommend">
    <div v-for="(index,pos) in result" :key="pos">
      <el-card :body-style="{ padding: '3px' }" class="table" @click.native="open(index,pos)">
        <el-image
          v-start="{ wp_id: index.id, u_id: getId}"
          style="width: 100%; height: 100%" 
          :src="index.url" 
          lazy/>
        <div></div>
      </el-card>
    </div>
    <div id="light" class="white_content">
      <el-image 
      :src="curUrl" />
    </div> 
    <div id="fade" class="black_overlay">
      <i class="custom-icon-close el-icon-circle-close" @click="close()"></i>
      <i class="custom-icon-left el-icon-arrow-left" @click="previous()"></i>
      <i class="custom-icon-right el-icon-arrow-right" @click="next()"></i>
    </div> 
    <div id="set" class="setting">
      <i class="custom-icon el-icon-download" style="margin-left:20px" @click="download()"></i>
      <div class="set_paper">设为壁纸</div> 
      <i id="like" class="custom-icon el-icon-star-off" style="margin-left:10px;" @click="addlike()"
      :class="isFavorate === true? 'custom-icon-font': ''">
      </i> 
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      result: [],
      u_id: -1,
      result: [],
      curUrl: '',
      curPos: 0,
      isFavorate: false
    }
  },
  methods: {
    open(index,pos) {
      this.curUrl = index.url
      this.curPos = pos
      this.isFavorate = this.result[this.curPos].favorate
      document.getElementById('light').style.display='block'
      document.getElementById('fade').style.display='block'
      document.getElementById('set').style.display='block'
    },
    close() {
      document.getElementById('light').style.display='none'
      document.getElementById('fade').style.display='none'
      document.getElementById('set').style.display='none'
    },
    next() {
      if(this.curPos !== this.result.length-1){
        this.curPos ++
        this.curUrl = this.result[this.curPos].url
        this.isFavorate = this.result[this.curPos].favorate
      }
    },
    previous () {
      if(this.curPos !== 0){
        this.curPos --
        this.curUrl = this.result[this.curPos].url
        this.isFavorate = this.result[this.curPos].favorate
      }
    },
    addlike() {
      if(this.u_id !== -1){
        this.axios.post('/api/addPaper', {
          wp: this.result[this.curPos].wp,
          u_id: this.u_id,
          wp_id: this.result[this.curPos].id,
          wp_url: this.result[this.curPos].url
        }).then((res) => {
        // success is 0 execute delete
        // success is 1 execute add
          if(res.data.success === '1'){
            // document.getElementById('like').style.color = 'red'
            this.result[this.curPos].favorate = true
            this.isFavorate = true
          }else{
            // document.getElementById('like').style.color = 'black'
            this.result[this.curPos].favorate = false
            this.isFavorate = false
          }
        }).catch((err) => {
          console.log(err)
        })
      } else {
        this.$notify({
          title: '失败',
          message: '请先登录',
          type: 'warning',
          duration: 2000
        })
      }
    },
    download() {
      if(this.u_id === -1) {
        this.$notify({
          title: '失败',
          message: '请先登录',
          type: 'warning',
          duration: 2000
        })
      } else {
        this.axios.post('/api/download', {
          u_id: this.u_id,
          id: this.result[this.curPos].id,
          wp_url: this.curUrl
        }).then((res) => {
          if(res.data.success === 1){
            var elemIF = document.createElement("iframe");
            elemIF.src = this.curUrl;
            elemIF.style.display = "none";
            document.body.appendChild(elemIF);
            this.$notify({
              title: '成功',
              message: '下载成功',
              type: 'success',
              duration: 2000
            })
            // var a = document.createElement('a')
            // a.download = name  || 'pic'
            // a.href = this.curUrl
            // a.click();
        }
      }).catch((err) => {
        console.log(err)
      })
      }
    }
  },
  mounted: function() {
    this.axios.post('/api/getRec', {
      u_id: this.$store.state.u_id
    }).then((res) => {
      console.log(res.data[0])
      for(let d in res.data) {
        var m = JSON.parse(res.data[d])
        this.result.push({
          id: m.wp_id,
          url: m.wp_url
        })
      }
      console.log(this.result)
    }).catch((err) => {
      console.log(err)
    })
  },
  computed: {
    getId() {
      var id = this.$store.state.u_id
      this.u_id = id
      // console.log(this.u_id)
      return id
    }
  }
}
</script>
<style>
#recommend {
  height: 600px;
  /* width: 100%; */
  overflow: scroll;
}
.table {
  width: 200px;
  height: 128px;
  margin-top: 30px;
  margin-left: 30px;
  float: left;
  padding: 0px;
}
.black_overlay {
  display: none;
  position: absolute;
  top: 0%;
  left: 0%;
  width: 100%;
  height: 100%;
  background-color: black;
  z-index:1001; 
  -moz-opacity: 0.8; 
  opacity:.70;  
  filter: alpha(opacity=80);
}  
.white_content {  
  display: none;  
  position: absolute;  
  top: 10%;  
  left: 25%;
  width: 50%;
  height: 100%;  
  /* padding: 16px;   */
  /* background-color: white;   */
  z-index:1002;  
}
.setting {
  display: none;  
  height: 45px;
  width: 200px;
  background-color: white;
  opacity: .70;
  position: fixed;
  bottom: 100px;
  left: 400px;
  z-index: 1003;
  border-radius: 22px;  
}
.custom-icon {
  font-size: 25px;
  float: left;
  margin-top: 10px;
  color: black;
}
.set_paper {
  height: 30px;
  width: 90px;
  background-color: black;
  opacity: 0.20;
  font-size: 17px;
  float: left;
  margin-top: 7px;
  margin-left: 10px;
  color: white; 
  border-radius: 10px;
  text-align: center;
  line-height: 30px;
}
.custom-icon-close {
  font-weight: 400;
  font-size: 3rem;
  float: right;
  margin-top: 30px;
  margin-right: 30px;
  color: white;
}
.custom-icon-left {
  font-size: 3rem;
  float: left;
  margin-left: 30px;
  margin-top: 400px;
  color: white;
}
.custom-icon-right {
  font-size: 3rem;
  float: right;
  margin-right: -40px;
  margin-top: 400px;
  color: white;
}
.custom-icon-font{
  color: red
}
</style>