<!--  -->
<template>
  <div id="chessboard-box" >
    <h1>{{title}}{{opera_num}}</h1> 
    <div id="chessboard" :style="checssboard_size_obj">
        <i class="cell" :style="cell_size_obj" :class="cell.classObj" @click="playChess(cell)" v-for="(cell,index) in cells"></i>
    </div> 
  </div>
</template>

<script>
export default {
  name: 'checssboard',
  data () {
    return {
      title: 'Vue ',
      cols:5,  //棋盘列数
      rows:5,  //棋盘行数
      cell_size:50, //棋盘格子的宽度和高度
      cells: {}, //棋盘所有格子
      cells_num: 0, //棋盘所有格子的数目
      opera_num: 0, //操作的步数,这里暂时是用来刷新页面
      success_num: 5,//同方向有五个相同类型的棋子，就算获胜
      currUser:{type:1,color:'white'},//当前用户棋子，默认白子
      users: {
        '1': {
          type: 2,
          color: 'black'
        },
        '2': {
          type: 1,
          color: 'white'
        }
      },
      winListP:[],//对人  的五元组
      winListC:[]//对电脑的 五元组
    }
  },
  computed: {
    //棋盘尺寸
    checssboard_size_obj () {

      return {
        width: this.cols * this.cell_size + 'px',
        height: this.rows * this.cell_size + 'px'
      }
    },
    //棋盘格子尺寸
    cell_size_obj () {

      return {
        width: this.cell_size + 'px',
        height: this.cell_size + 'px'
      }
    }
  },
  methods: {
    init () {

      this.initCells();
      this.initWinList();
    },
    //下棋
    playChess (cell){

      if(cell.type == 0){

        cell.type = this.currUser.type;
        cell.classObj['cell-' + this.currUser.color] = true;
        this.currUser = this.users[this.currUser.type];
        this.opera_num ++;

        if(this.isSuccess(cell)){
          alert('game over');
          window.location.reload();
        }
      }
    },
    isSuccess (cell) { //检查是否有人胜利

      var _self  = this;
      var h_num  = this.horizontal(cell);
      var v_num  = this.vertical(cell);
      var ld_num = this.leftUpToRightDown(cell);
      var rd_num = this.rightUpToLeftDown(cell);
      var res = [h_num,v_num,ld_num,rd_num].map(function(num,index){
        
        return num >= _self.success_num;
      }).join(',');
      
      if(res.indexOf('true') >= 0){
        return true;
      }
      return false;
    },
    //检查水平方向
    horizontal (cell) {
      var num = 1;
      for (var i = 1; i <= this.success_num; i++) {
        var leftIndex = cell.index - i;
        if(Math.floor(leftIndex / this.cols) != cell.y){ //保证在同一行
          break;
        } 
        if(this.cells[leftIndex].type == cell.type){
          num++;
        }else{
          break;
        }
      }

      for (var i = 1; i <= this.success_num; i++) {
        var rightIndex = cell.index + i;
        if(Math.floor(rightIndex / this.cols) != cell.y){
          break;
        }
        if(this.cells[rightIndex].type == cell.type){
          num++;
        }else{
          break;
        }
      }
      return num;
    },
    //垂直方向
    vertical (cell) {
      var num = 1;
      for (var i = 1; i <= this.success_num; i++) {
        var upIndex = cell.index - i * this.cols; 
        if(upIndex < 0){
          break;
        }
        if(this.cells[upIndex].type == cell.type){
          num ++;
        }else{
          break;
        }
      }
      for (var i = 1; i <= this.success_num; i++) {
        var downIndex = cell.index + i * this.cols;
        if(downIndex > this.cells_num - 1){ 
          break;
        }
        if(this.cells[downIndex].type == cell.type){
          num ++;
        }else{
          break;
        }
      }

      return num;
    },
    //左上到右下的方向
    leftUpToRightDown (cell) { 
      var num = 1,//代表当前棋子
          leftUpIndex,
          rightDownIndex;
      for (var i = 1; i <= this.success_num; i++) { //左上
        leftUpIndex = cell.index - i * (this.cols + 1) ;
        if(leftUpIndex < 0){
          break;
        }
        if(this.cells[leftUpIndex].type == cell.type){
          num++;
        }else{
          break;
        }
      }
      for (var i = 1; i <= this.success_num; i++) { //右下
        rightDownIndex = cell.index + i * (this.cols + 1);
        if(rightDownIndex > this.cells_num - 1){
          break;
        }
        if(this.cells[rightDownIndex].type == cell.type){
          num++;
        }else{
          break;
        }
      }
      return num;
    },
    //右上到左下的方向
    rightUpToLeftDown (cell) {
      var num = 1,
          rightUpIndex,
          leftDownIndex;
      for (var i = 1; i <= this.success_num; i++) {
        rightUpIndex = cell.index - (this.cols - 1) * i;
        if(rightUpIndex < 0){
          break;
        }
        if(this.cells[rightUpIndex].type == cell.type){
          num++;
        }else{
          break;
        }
      }

      for (var i = 1; i <= this.success_num; i++) {
        leftDownIndex = cell.index + (this.cols - 1) * i;
        if(leftDownIndex > this.cells_num - 1){
          break;
        }
        if(this.cells[leftDownIndex].type == cell.type){
          num++;
        }else{
          break;
        }
      }

      return num;
    },
    //初始化棋盘格子
    initCells () {
      var vm = this;
      this.cells_num = this.cols  * this.rows;
      for (var i = 0; i <this.cells_num; i++) {

        var x = i % this.cols;
        var y = Math.floor(i / this.cols);
        this.cells[i] = {
          index: i,
          x: x,
          y: y,
          type: 0, //0：代表没字，1：代表白子，2:代表黑子
          classObj:{
            'cell-white':false,
            'cell-black':false,
            'cell-no-border': x == y && x == 0 ? true : false,
            'cell-one-col': x == 0 && y > 0 ? true : false,
            'cell-one-row': y == 0 && x > 0? true : false
          }
        }
      }
    },
    //初始化 五元组 能连城一条线的五个格子的集合
    initWinList () { 

      var _self = this,
          winList = [],
          index,
          minIndex,
          maxIndex;
      //横向连接
      Object.values(this.cells).forEach(function(cell,i){

        index = cell.y * _self.cols + cell.x; //计算出 当前格子对应的索引
        if(Math.floor( (index + 4) / _self.cols) == cell.y){ //保证在同一行

          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index + i;
            })
          });
        }
        //竖直方向连接
        maxIndex = index + _self.cols * 4;
        if( maxIndex % _self.cols == cell.x && maxIndex < _self.cells_num ){ //保证是在同一列

          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index + _self.cols * i;
            }) 
          })
        }

        //斜方向 左下
        maxIndex = index + (_self.cols + 1) * 4;
        //最大索引不能超出棋盘最大数，Y轴坐标 - 当前Y坐标 <= 4
        if(maxIndex < _self.cells_num && Math.floor(maxIndex / _self.cols) - cell.y <= 4){ 

          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index + (_self.cols + 1) * i;
            })
          })
        }

        //斜方向 右上
        minIndex = index - (_self.cols - 1) * 4;
        //最小索引不能小于 0 ，X轴坐标必须大于当前X轴坐标
        if(minIndex >= 0 &&  minIndex % _self.cols > cell.x){
          // debugger
          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index - (_self.cols - 1) * i;
            }),
            status: 1,//状态，标识是否可用,0:表示不可以用，1：表示可用
          })
        }


      })
      
      this.winListC = winList;
      this.winListP = winList.concat([]);
      
    },
    //电脑下棋
    aiPlayChess () {

      //在winListC中搜索有效的坐标，并计算每个坐标的分值 返回最大分值 --- 进攻
      //在winlistP中搜素有效坐标，并计算每个坐标的分值，返回最大的分值 --- 防守
      //同等分值下 进攻优先
      
      var c_cell = this.getPlayCell(this.winListC);
      var p_cell = this.getPlayCell(this.winListP);

      if(c_cell.score >= p_cell.score){

        this.playChess(c_cell);
      }else{

        this.playChess(p_cell);
      }
    },

    //返回最佳下棋的格子
    //{
    //  index:  0,//格子的索引
    //  score: 100,//最大分值
    //}
    getPlayCell (winList) {

      var _self = this;
      var resCell = {},//返回的cell对象
          maxScore = 0,
          score = 0; 
      winList.forEach(function(val){

        if(val.status == 1){ //状态可用

          val.indexs.forEach(function(index){
            score = _self.getScoreByIndex(index);
            if(score > maxScore){
              maxScore = score; //记录最大score
              resCell = _self.cells[index]; //记录单元格
            }
          })
        }
      })

      //保存最大分值
      resCell.score = maxScore;

      return resCell;
    },
    //根据单元格索引，获得单元格的分值
    getScoreByIndex (index) {

      var score = 0;
      var h_num = this.horizontal(this.cells[index]),//如果电脑下在该位置，则水平方向上的连子数
          v_num = this.vertical(this.cells[index]),//垂直方向的连子数
          ld_num = this.leftUpToRightDown(this.cells[index]),//左上到右下的连子数
          rd_num = this.rightUpToLeftDown(this.cells[index]);//右上到左下的连子数
      [h_num,v_num,ld_num,rd_num].map(function(num,i){
        switch(num){
          case 0 : score += 0;break;
          case 1 : score += 10;break;
          case 2 : score += 100;break;
          case 3 : score += 1000;break;
          case 4 : score += 10000;break;
          case 5 : score += 100000;break;
        }
      })

      return score;
    },
    //每次下完棋后重置 winlist，把依赖某个点，且改点不是本方棋子的所有集合都删除
    resetWinList () {

      var _self = this;
      this.winListC.forEach(function(val,i){

        if(val.status == 1){

          val.indexs.forEach(function(index){

            if(_self.cells[index].type == 1){ //1代表白子，用户先下
              _self.winListC[i].status = 0; //跟新五元组 依赖的坐标已被 用户下了，所以该坐标失效
              return;
            }
          })
        }
        
      })

      this.winListP.forEach(function(val,i){

        if(val.status == 1){

          val.indexs.forEach(function(index){

            if(_self.cells[index].type == 2){ //2代表黑子，电脑已经下了
              _self.winListP[i].status = 0; //跟新五元组 依赖的坐标已被 电脑下了，所以该坐标失效
              return;
            }
          })
        }
      })

    }
  },
  created  () {

    this.init();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
*{
  padding: 0px;
  margin: 0px;
}
#chessboard{
  display: flex;
  flex-wrap: wrap;
  margin: 0px auto;
  background-image: url(../assets/bg.png);
  position: relative;
}
.cell{
  display: inline-block;
  box-sizing: border-box;
  border-radius: 50%;
  position: relative;
}
.cell-white{
  background: white;
}
.cell-black{
  background: black;
}
.cell:after{
  width: 100%;
  height: 100%;
  position: relative;
  left: -50%;
  top:-50%;
  content:' ';
  display: inline-block;
  border-right: 1px solid black;
  border-bottom: 1px solid black;
  pointer-events:none;
}
.cell-no-border:after{
  border: 0px;
}
.cell-one-col:after{
  border: 0px;
  border-right: 1px solid black;
}
.cell-one-row:after{
  border: 0px;
  border-bottom: 1px solid black;
}
</style>
