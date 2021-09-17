<template>  
  <div class="home">
    <div class="main">
      <div class="shadow"  :style="{backgroundImage:`url(${song_Info.img_url})`}"></div>
      <div class="content_wrapper">
        <div class="title_wrapper">
          {{song_Info.name}}
          <div class="search_input">
            <input 
             type="text" 
             placeholder="请输入歌曲名" 
             v-model="search_song" 
             @keyup.enter="searchSong"
             @input="inputListening"
             @blur="onblur"
             >
          </div>
        </div>
        <div class="img_wrapper">
          <div class="img" :style="{backgroundImage:`url(${song_Info.img_url})`}"></div>
        </div>
        <div class="progress_wrapper">
          <div class="progress_bar" ref="progress_bar">
            <span>{{changeTime(currTime)}}</span>
            <div class="progress" ref="progress">
              <div class="bar" 
                   :style="{width: barWidth +'px'}"
                   >
                <div class="after" ref="after"
                   @touchstart="mousedown"
                   @touchmove="mousemove"
                   @touchend="mouseup"
                   >
                </div>   
            </div>
            </div>
            <span>{{changeTime(totalTime)}}</span>
          </div>
          <div class="controls">
            <div class="button prev" @click="prev_">
                <i class="iconfont icon-houtui"></i>              
            </div>
            <div 
              class="button play" 
              >
                <i class="iconfont icon-bofang" v-if="isshow" @click="onPlay"></i>
                <i class="iconfont icon-zantingtingzhi" v-else @click="onPause"></i>
              </div>
            <div class="button next" @click="next_">
                <i class="iconfont icon-qianjin"></i>
            </div>
          </div>
        </div>
        <audio :src="song_Info.song_url"  ref="audio" ></audio>
        <div class="down" :class="{downActive:isdown}">
          <ul v-if="searchReplayData">
            <template v-for="(item,index) in searchReplayData" >
              <li :key="item+index" @mousedown.stop="chooseSong(index)"  ref="li">
              {{item.name}}
              <div class="singName">
               {{item.singer}}
              </div>
            </li>
            </template>
          </ul>
        </div>
        <div class="playlist_btn">
          <i class="iconfont icon-bofangliebiao" 
          @click="playSong"></i>
        </div>
        <div class="play_list" :class="{playActive:isActive}">
          <ul v-if="playList">
            <template v-for="(item,index) in playList" >
              <li :key="item+index" @mousedown.stop="choosePlayLostSong(index)"  ref="li">
              {{item.name}}
            </li>
            </template>
          </ul>
        </div>
      </div>
    </div>  
  </div>
</template>

<script>
let _self = null;
    // 防抖
      function debounce(fn,delay) {
          let timer = null;
          return function(){
            let conxt = this;
            let args = arguments;
            if(timer) {
            clearTimeout(timer)
          } 
          timer = setTimeout(()=>{
            fn.apply(conxt,args)
          },delay)
        }
      }
export default {
  data() {
    return{
      song_Info:{
          name:'海阔天空',
          img_url:'https://p1.music.126.net/zZtUDuWk6qIe3ezMt4UMjg==/109951165796417308.jpg?param=130y130',
          song_url:"https://link.hhtjim.com/163/347230.mp3",
          song_num:'0',
        },
      currTime: 0,
      totalTime: 0,
      barWidth:0,
      isplay: true,
      isshow: true,
      isplaylist: '',
      count: 0, 
      search_song: '',
      searchReplayData:[],
      searchData_list:[],
      isdown:true,
      playList:[
      ],
      isActive: true,
    }
  },
  created() {
    _self = this;
  },
  mounted() {
    this.init()  
  },
  methods: {
    init() {
      let audio = this.$refs.audio
      audio.addEventListener('canplay',()=>{
      this.totalTime=0
      if(!this.totalTime) {
        this.totalTime = Math.floor(audio.duration)
      }
      }),
      audio.addEventListener('timeupdate',()=>{
          this.onUpdate()
      })
    },
    onPlay() {
      this.$nextTick(()=>{
        this.$refs.audio.play();
        this.onUpdate();
        this.isplay =false;
        this.isshow = false;
        if(this.playList.length === 0) {
          this.addPlayList()
        }else{
              for(let i in this.playList) {
                if(this.playList[i].name == this.song_Info.name){
                  return console.log('歌曲已经在播放列表中了111')
                }
              }
              this.addPlayList()
          }
        })
    },
    onPause() {
        this.$refs.audio.pause();
        this.isplay=true;
        this.isshow = true;
    },
    onUpdate() {
      this.currTime = Math.floor(this.$refs.audio.currentTime)
      this.setBarWidth()
    },
    setBarWidth() {
      const progress = this.$refs.progress
      if(!progress) return 0
      const allWidth = progress.clientWidth
      const percent = (this.currTime/this.totalTime)
      this.barWidth = allWidth * percent
    },
    mousedown() {
      this.onPause()
    },
    mousemove(event) {
      let eventOn = event.touches[0].clientX
      let audio = this.$refs.audio
      const progress = this.$refs.progress
      if(!progress) return 0
      const allWidth = progress.clientWidth
      if(eventOn >=85 && eventOn < 315) {
       this.barWidth = eventOn - 85;
       audio.currentTime = this.totalTime*this.barWidth /allWidth
      }
    },
    mouseup() {
      if(this.isplay) {
        this.onPause()
      }else{
        this.onPlay()
      }
    },
    prev_() {
     let play_num = '';
     if(this.playList.length == 0) {
        console.log('播放列表为空')
        }else{
          for(let i in this.playList) {
            if(this.playList[i].name === this.song_Info.name ){
               play_num = i;
            }
          } 
              if(play_num == 0) {
                this.song_Info = this.playList[this.playList.length-1];
             } else {
                this.song_Info =  this.playList[+play_num-1]          
             }
        }
      
      this.$nextTick(()=>{
        if(!this.isplay) {
        this.$refs.audio.play();
        this.onUpdate();
        this.isplay =false;
        this.isshow = false;
      }else{
        this.$refs.audio.pause();
        this.onUpdate();
        this.isplay =true;
        this.isshow = true;
      }
      })
    
    },
    next_() {
      let play_num = '';
     if(this.playList.length == 0) {
        console.log('播放列表为空')
        }else{
          for(let i in this.playList) {
            if(this.playList[i].name === this.song_Info.name ){
               play_num = i;
            }
          } 
            if(play_num == this.playList.length-1) {
                this.song_Info = this.playList[0];
             } else {
                this.song_Info = this.playList[+play_num + 1]          
             }
        }
      this.$nextTick(()=>{
        if(!this.isplay) {
        this.$refs.audio.play();
        this.onUpdate();
        this.isplay =false;
        this.isshow = false;
      }else{
        this.$refs.audio.pause();
        this.onUpdate();
        this.isplay =true;
        this.isshow = true;
      }
      })
    },
    // 监听输入事件
    inputListening: debounce(()=>{
        if(!_self.search_song) {
          _self.isdown = true;
        }else {
          return 
        }
      },200),
    // 获取搜索列表
    searchSong() {
      this.axios.get('https://autumnfish.cn/search?keywords='+this.search_song).then((response)=>{
          let tags = response.data.result.songs;
          let objArray = this.searchReplayData;
          for ( let index in tags) {
            let obj = {};
            obj.num = +index+1;
            obj.albumId = tags[index].album.id
            obj.songId = tags[index].id;
            obj.name = tags[index].name;
            obj.singer = tags[index].artists[0].name
            objArray.push(obj)
          }
          for ( let i in objArray) {
            objArray[i].songurl = 'https://link.hhtjim.com/163/'+objArray[i].songId+'.mp3';
            objArray[i].album_imgurl = 'https://autumnfish.cn/playlist/cover/update?id='+objArray[i].albumId;
          }
          this.isdown= false;
        })
    },
    // 获取歌曲封面
    set_albumImg() {
      this.axios.get(this.searchReplayData.album_imgurl).then((e)=>{
        console.log(e)
      })
    },
    onblur() {
      if(this.search_song){
        this.isdown=true;
      }    
      this.searchReplayData = [];    
    },
    chooseSong(index) {
      let searchReplayData = this.searchReplayData
      let song_Info = this.song_Info;
      song_Info.name = searchReplayData[index].name;
      song_Info.song_url = searchReplayData[index].songurl;
      song_Info.song_num = searchReplayData[index].num;
      if(this.playList.length === 0) {
        this.addPlayList()
      }else {
        for(let i in this.playList) {
          if(this.playList[i].name == song_Info.name) {
            console.log('歌曲已经在播放列表中了222')
            return this.song_Info = this.playList[i];  
          }
        }
        this.addPlayList()
      }
       this.$nextTick(()=>{
         if(!this.isplay) {
          this.$refs.audio.play()
          this.onUpdate();
          this.isplay = false;
          this.isshow = false;
        }else {
          this.$refs.audio.pause()
          this.onUpdate();
          this.isplay = true;
          this.isshow = true;
        }
       })
      
    },
    // 转换时间
    changeTime(sec) {
      let s = `${sec%60}`.padStart(2,'0');
      let min = `${Math.floor(sec/60)}`.padStart(2,'0');
      return min+':'+s;
    },
    //播放列表
    addPlayList() {
      // if( this.playList.length == 0){
        let obj = {};
        obj.name = this.song_Info.name;
        obj.song_url = this.song_Info.song_url;        
        obj.img_url = this.song_Info.img_url;
        this.playList.push(obj);
      // }else{
      //   let obj = {};
      //   obj.name = this.song_Info.name;
      //   obj.song_url = this.song_Info.song_url;        
      //   obj.img_url = this.song_Info.img_url;
      //       this.playList.push(obj);
      // }
    },
    //播放按钮
    playSong() {
      if(this.playList.length == 0) {
        console.log('播放列表暂时没有歌曲噢')
      }else {
        this.isActive = !this.isActive;
      }
    },
    //选择播放列表中的歌曲 
    choosePlayLostSong(index) {
      let playListOn = this.playList;
      let song_InfoOn = this.song_Info;
      song_InfoOn.name = playListOn[index].name;
      song_InfoOn.song_url = playListOn[index].songurl;
      song_InfoOn.song_num = playListOn[index].num
      if(!this.isplay) {
        this.onPlay()
      }else {
        this.onPause()
      }
      this.isActive = true
    },
  }
}
</script>

<style lang="less" scoped>
.main {
  width: 100vw;
  height: 100vh;
  position: relative;
}
.shadow{
  width: 100%;
  height: 100%;
  background-position: 50% 50%;
  background-size: 100%;
  filter: blur(120px);
}
.content_wrapper{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display:flex;
  flex-direction: column;
  justify-content: space-around;
}
.title_wrapper{
  width: 100%;
  height: 40px;
  line-height: 40px;
  text-align: center;
  color:  #fff;
  font-size: 20px;
  position:relative;
  .search_input{
    position: absolute;
    top: 0;
    right: 5px;
    input{
      outline: none;
      border: none;
      width: 90px;
      height: 24px;
      // border-radius: 5px;
      padding-left: 10px;
      &::placeholder{
        color: gray;
      }
    } 
  }
}
.img_wrapper {
  position: relative;
  height: 400px;
}
.img{
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
  width: 200px;
  height: 200px;
  background-color: #fff;
  background-position: 50% 50%;
  background-size: 100%;
  border-radius: 50%;
  border: 30px solid #333;
}
.progress_wrapper{
  width: 100%;
  height: 150px;
}
.progress_bar{
  display: flex;
  justify-content: space-around;
  align-items: center;
  span {
    display: inline-block;
    text-align: center;
    width: 43px;
    height: 100%;
  }
}
.progress{
  display: inline-block;
  width: 230px;
  height: 3px; 
  background-color: cadetblue;
  opacity: .5;

}
.bar{
  position:relative;
  height: 3px; 
  background-color: #fff;
  opacity: .5;
}
.after{
  position: absolute;
  top: 1px;
  left: 100%;
  transform: translate(-50%,-50%);
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: #fff;
}
.controls{
  display: flex;
  justify-content: space-around;
  align-items: center;
  padding-top: 20px;
  .button {
    width: 30px;
    height: 30px;
    border-radius: 20px;
    border: 1px solid violet;
    position: relative;
    i {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%,-50%);  
      font-size: 20px;
    }
  }
  .play {
    width: 50px;
    height: 50px;
    border-radius: 30px;
    border: 1px solid violet;
    position: relative;
    i {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%,-50%);  
      font-size: 40px;
    }
    .active{
      display: none;
    }
  }
}
.down {
  display: block;
  position: absolute;
  top: 49px;
  right: 5px; 
  max-width: 300px;
  max-height: 200px;
  background-color: #fff;
  overflow: hidden;
  overflow-y:auto;
  ul {
    margin: 0;
    padding: 0 7px 0 7px;
    border: 1px solid #fff;
    // width: 200vw;
    li{
      display: inline-block;
      width: 270px;
      list-style: none;
      text-align: left;
      font-size: 17px;
      color: gray;  
      padding-top: 2px;
      padding-bottom: 2px;
      .singName{
        float: right;
      }
    }
  }
}
.downActive {
  display: none;
}
.play_list {
  display: block;
  position: absolute;
  bottom: 210px;
  right: 22px; 
  width: 100px;
  max-height: 200px;
  background-color: black;
  opacity: 0.5;
  overflow: hidden;
  overflow-y:auto;
  border-radius: 23px;
  padding: 6px 5px 6px 5px;
  text-align: center;
  ul {
    margin: 0;
      list-style: none;
      margin:0;padding:0;
    li{
      padding: 0 7px 0 7px;
      display: block;
      list-style: none;
      text-align: left;
      font-size: 17px;
      color: wheat;  
      padding-top: 2px;
      padding-bottom: 2px;
      overflow: hidden;
      text-overflow:  ellipsis;
      white-space: nowrap;
    }
  }
}
.playlist_btn{
  display: inline-block;
  position: absolute;
  bottom: 182px;
  right: 43px;
  .iconfont {
    font-size: 32px;
    color: steelblue;
  }
}
.playActive {
  display: none;
}
</style>
