<template>
    <div>
        <div class="baseinfo_box">
            <div class="avatar">
                <el-image :src="userInfo.avatarUrl">
                    <div slot="error" class="image-slot">
                        <i class="el-icon-picture-outline"></i>
                    </div>
                </el-image>
            </div>
            <div class="base_form">
                <el-form>
                    <el-form-item>
                        {{userInfo.nickname}} 
                        <el-divider direction="vertical"></el-divider>
                        {{userInfo.gender==1?'男':'女'}}</el-form-item>
                    <el-form-item>xxx</el-form-item>
                    <el-form-item>个人介绍: {{userInfo.signature}}</el-form-item>
                </el-form>
            </div>
            <div class="clear"></div>
        </div>
        <div class="container">
            <section>
                <div class="table-top">
                    <h3>最近听歌</h3> 共{{trackCount}}首
                </div>
                
                <el-table
                :data="tracks"
                empty-text="暂无数据"
                stripe
                style="width: 100%"

                >
                <el-table-column prop="data.name" label="歌曲标题" width="220">
                </el-table-column>
                <el-table-column prop="data.ar[0].name" label="歌手" width="220">
                </el-table-column>
                <el-table-column prop="playTime" label="播放时间" width="220">
                </el-table-column>
                <el-table-column prop="data.al.name" label="专辑" width="220">
                </el-table-column>
                <el-table-column>
                    <template slot-scope="scope">
                        <div  class="action">
                            <div v-if="(likedSongs.indexOf(scope.row.data.id) == -1 )">
                                <img @click="likeSong(scope.row,true)"
                                    class="icon" 
                                    src="../../assets/like0.svg"
                                />
                            </div>
                            <div v-else>
                                <img @click="likeSong(scope.row,false)"
                                    class="icon" 
                                    src="../../assets/like1.svg"
                                />
                            </div>
                            <div>
                                <img
                                    @click="getSongUrl(scope.$index, scope.row)"
                                    class="icon"
                                    src="../../assets/bofang.svg"
                                />
                            </div>
                        </div>
                    </template>
                </el-table-column>
                </el-table>
            </section>
            <section>
                <div class="table-top">
                    <h3>我的歌单</h3>（{{subPlaylistCount}}）
                </div>
                <el-row>
                    <el-col :span="4"  v-for="(item,index) in playlist" :key="index">
                        <div class="imgbox">
                            <el-image :src="item.coverImgUrl" @click="toPlaylistDetail(item.id,index)" lazy></el-image>
                            <div class="listname">{{item.name}}</div>
                            <div class="playcount"><i class="el-icon-headset"></i> {{item.playCount}} <i class="el-icon-video-play right"></i></div>
                        </div>
                    </el-col>
                </el-row>
                <!-- <el-pagination hide-on-single-page background layout="prev, pager, next" :total="1000">
                </el-pagination> -->
            </section>     
        </div>
    </div>
</template>

<script>

export default ({
    data(){
        return{
            userInfo:{},
            listenSongs: 0,
            gender:'',
            avatarUrl:'',
            signature:'',
            followeds:0,
            follows:0,
            subPlaylistCount:0,
            trackCount:0,
            tracks: [],
            playlist:[],
            playlistid:[],
            likedSongs:[]
        }      
    },
    created(){
        this.getUserInfo()
        this.getSongMV()
        this.getRecommendSongs()
        this.getRecentSongs()
        this.getPlaylist()
    },
    methods:{
        // 获取用户基本信息

        async getUserInfo(){
            const {data:res} = await this.$http.get('/user/account')
            console.log(res)
            this.userInfo = res.profile
            console.log(this.userInfo)
        },
        
        async getSongMV(){
            const {data:res} = await this.$http.get('/user/subcount?cookie='+window.sessionStorage.getItem('cookie'));
            // console.log(res)
            this.subPlaylistCount = res.subPlaylistCount + res.createdPlaylistCount
        },
        
        async getRecommendSongs(){
            const {data:res} = await this.$http.get('/recommend/songs?cookie='+window.sessionStorage.getItem('cookie'));
            // console.log(res)
        },

        async likeSong(row,like){
            row = row.data
            let id = row.id
            // 发起 喜欢音乐 请求
            const {data:res} = await this.$http.get('/like?like='+like+'&id='+id+'&cookie='+window.sessionStorage.getItem('cookie'))
            // 返回 200 还有添加到的歌单的id
            if(res.code==200){
                this.$message({ message: "操作成功!",type: 'success',offset:100 });
            }else{
                this.$message({ message: "操作失败!",type: 'success',offset:100 });
            }
        },
        
        async islike(rids){
            const { data: res } = await this.$http.get("/likelist?id=" + window.sessionStorage.getItem('uid')+'&cookie='+window.sessionStorage.getItem('cookie'));
            // 对比两个数组 得到相同的id
            // 然后赋值给一个数组变量
            // 再判断上面的某id是否属于这个数组里面

            function getSame2(arr1,arr2){
                let arr3=arr1.filter(item=>{
                    return arr2.includes(item)
                })
                return arr3
            }

            function getSame(arr1, arr2) {
                return [...new Set(arr2)].filter(item => 
                    arr1.includes(item)
                )
            }  
            this.likedSongs=getSame2(rids,res.ids)
        },

        async getRecentSongs(){
            const {data:res} = await this.$http.get('/record/recent/song?cookie='+window.sessionStorage.getItem('cookie'));
            this.trackCount = res.data.total
            for(let i=0;i<res.data.total;i++){
                res.data.list[i].playTime = new Date(res.data.list[i].playTime);
                res.data.list[i].playTime=(res.data.list[i].playTime.toString()).substring(16,24)
            }
            let rids = []
            for(let i=0;i<res.data.list.length;i++){
                rids[i] = res.data.list[i].data.id
            }
            this.islike(rids)
            // 思路2: 这里返回出来一组index，将相同歌曲的trakcs多一个字段，表示是否喜欢
            // 然后上面可以直接用这个字段
            // 我们再用一个数组变量去 直接监听，点击就变，false true。
            // v-if那里， 如果两个变量（一个是tracks的某字段，一个是数组变量）；如果两个都相同，就不变，若两个不同，就变成第二个的样子
            // img里面那个要点击的时候才会出触发事件。
            this.tracks = res.data.list.slice(0,5)
            console.log(this.tracks)
        },

        async getPlaylist(){
            const {data:res} = await this.$http.get('/user/playlist?limit=27&uid='+window.sessionStorage.getItem('uid'));
            this.playlist = res.playlist
        },

        // 跳转页面;这里的id是歌单的id，然后传过去
        toPlaylistDetail(id,index){
            console.log(id)
            this.$router.push({
                path:"/album",
                query: {   
                    id: id,
                    index:index
                } 
            })
        },

        async getSongUrl(index, row) {
            row=row.data
            let id = row.id;
            let name = row.name;
            // 歌手名字 歌手id
            let ar = row.ar[0].name;
            let ar_id = row.ar[0].id

            let al_picUrl = row.al.picUrl
            //  检查音乐是否可播放  
            try {
                // 第一个请求只是看是否能请求成功，不会用到变量
                let check= await this.$http.get("/check/music?id=" + id);
                this.toLyrics(id,name,ar,al_picUrl)
            }catch (error) {
                console.log(error);
                this.$message({ message: "亲爱的，暂无版权!" });
                }
            },

        toLyrics(id,name,ar,al_picUrl){
            this.$router.push({
                path:"/lyrics",
                query: {   
                    id: id,
                    name:name,
                    ar:ar,
                    al_picUrl:al_picUrl
                } 
            })
        }
    }
})
</script>

<style lang="less" scoped>
.baseinfo_box{
    .avatar{
        float: left;
        width: 10rem;
        margin-right: 1rem;
        background-color:#fff;
        .el-image{
            margin: 8px;
        }
    }
    .base_form{
        float:left;
        text-align: left;
        width: 20rem;
        .el-form{
            padding:0 10px ;
            box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.05)
        }
    }
    .clear{
        clear: both;
    }
}

.container{
    .table-top{
        padding:10px 0;
    }
    h3{
        display: inline;
    }
    .el-table {
        .el-table-column {
        overflow: hidden;
        text-overflow: ellipsis; /* 溢出用省略号表示 */
        white-space: nowrap; /* 始终保持在一行显示 */
        }
    }
    .action{
        display: flex;
        .icon {
            width: 2rem;
            cursor: pointer;
        }
    }
    .el-row{
        .el-col{
            width: 200px;
            height: 230px;
            .imgbox{
                width: 180px;
                position: absolute;
                .el-image{
                    height: 180px;
                    cursor:pointer;
                }
                .listname{
                    overflow:hidden;
                    text-overflow:ellipsis;  /* 溢出用省略号表示 */
                    white-space:nowrap;  /* 始终保持在一行显示 */
                }
                .playcount{
                    background-color: lightcyan;
                    opacity: 0.8;
                    padding: 5px;
                    position:relative;
                    top: -3.7rem;
                }
            }         
        }    
    }
}
</style>
