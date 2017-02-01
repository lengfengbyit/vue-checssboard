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
//求和
Array.prototype.sum = function(){
  var result = 0;
  this.forEach(function(v){
    result += v;
  })
  return result;
}
//删除指定值得元素
Array.prototype.removeByVal = function(val){
  var res = [];
  for (var i = 0; i < this.length; i++) {
    if(this[i] != val){
      res.push(val);
    }
  }
  return res;
}
export default {
  name: 'checssboard',
  data () {
    return {
      title: 'Vue 五子棋',
      cols:16,  //棋盘列数
      rows:16,  //棋盘行数
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
      winList:[],
      scoreMap: {
        'blank': 7, //五元组为空
        'B':35,
        'BB':800,
        'BBB':15000,
        'BBBB':800000,
        'W':15,
        'WW':400,
        'WWW':1800,
        'WWWW':100000,
        'polluted':0 //五元组中黑白都有
      }
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
        this.resetWinList();

        if(this.isSuccess(cell)){
          if(cell.type == 1){
            alert('你赢了');
          }else{
            alert('电脑赢了');
          }
          window.location.reload();
        }else if(this.currUser.type == 2){

          this.aiPlayChess(); //电脑下棋
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
    //检查水平方向 flag:代表是否包换空格，判断权值的时候使用
    horizontal (cell,flag) {
      var num = 1;
      for (var i = 1; i <= this.success_num; i++) {
        var leftIndex = cell.index - i;
        if(Math.floor(leftIndex / this.cols) != cell.y){ //保证在同一行
          break;
        } 
        if(this.cells[leftIndex].type == cell.type || (flag && this.cells[leftIndex].type == 0)){
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
        if(this.cells[rightIndex].type == cell.type || (flag && this.cells[rightIndex].type == 0)){
          num++;
        }else{
          break;
        }
      }
      return num;
    },
    //垂直方向
    vertical (cell,flag) {
      var num = 1;
      for (var i = 1; i <= this.success_num; i++) {
        var upIndex = cell.index - i * this.cols; 
        if(upIndex < 0){
          break;
        }
        if(this.cells[upIndex].type == cell.type || (flag && this.cells[upIndex].type == 0)){
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
        if(this.cells[downIndex].type == cell.type || (flag && this.cells[downIndex].type == 0)){
          num ++;
        }else{
          break;
        }
      }

      return num;
    },
    //左上到右下的方向
    leftUpToRightDown (cell,flag) { 
      var num = 1,//代表当前棋子
          leftUpIndex,
          rightDownIndex;
      for (var i = 1; i <= this.success_num; i++) { //左上
        leftUpIndex = cell.index - i * (this.cols + 1) ;
        if(leftUpIndex < 0){
          break;
        }
        if(this.cells[leftUpIndex].type == cell.type || (flag && this.cells[leftUpIndex].type == 0)){
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
        if(this.cells[rightDownIndex].type == cell.type || (flag && this.cells[rightDownIndex].type == 0)){
          num++;
        }else{
          break;
        }
      }
      return num;
    },
    //右上到左下的方向
    rightUpToLeftDown (cell,flag) { 
      var num = 1,
          rightUpIndex,
          leftDownIndex; 
      for (var i = 1; i <= this.success_num; i++) {
        rightUpIndex = cell.index - (this.cols - 1) * i;
        if(rightUpIndex <= 0 || this.cells[rightUpIndex].x <= cell.x){
          break;
        }
        if(this.cells[rightUpIndex].type == cell.type || (flag && this.cells[rightUpIndex].type == 0)){
          num++;
        }else{
          break;
        }
      }

      for (var i = 1; i <= this.success_num; i++) {
        leftDownIndex = cell.index + (this.cols - 1) * i;
        if(leftDownIndex >= this.cells_num - 1 || this.cells[leftDownIndex].y <= cell.y){
          break;
        }
        if(this.cells[leftDownIndex].type == cell.type || (flag && this.cells[leftDownIndex].type == 0)){
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
    //初始化 五元组 能连城一条线的五个格子的集合(能赢的规则)
    initWinList () { 

      var _self = this,
          winList = [],
          index,
          minIndex,
          maxIndex;
      
      Object.values(this.cells).forEach(function(cell,i){
        //横向连接
        index = cell.y * _self.cols + cell.x; //计算出 当前格子对应的索引
        if(Math.floor( (index + 4) / _self.cols) == cell.y){ //保证在同一行

          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index + i;
            }),
          });
        }
        //竖直方向连接
        maxIndex = index + _self.cols * 4;
        if( maxIndex % _self.cols == cell.x && maxIndex < _self.cells_num ){ //保证是在同一列

          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index + _self.cols * i;
            }),
          })
        }

        //斜方向 左下
        maxIndex = index + (_self.cols + 1) * 4;
        //最大索引不能超出棋盘最大数，Y轴坐标 - 当前Y坐标 <= 4
        if(maxIndex < _self.cells_num && Math.floor(maxIndex / _self.cols) - cell.y <= 4){ 

          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index + (_self.cols + 1) * i;
            }),
          })
        }

        //斜方向 右上
        minIndex = index - (_self.cols - 1) * 4;
        //最小索引不能小于 0 ，X轴坐标必须大于当前X轴坐标
        if(minIndex >= 0 &&  minIndex % _self.cols > cell.x){
         
          winList.push({
            indexs: Array.apply(null,{length:_self.success_num}).map(function(_,i){
              return index - (_self.cols - 1) * i;
            }),
          })
        }
      })

      this.winList = winList;
    },
    //电脑下棋
    aiPlayChess () {

      //在winListC中搜索有效的坐标，并计算每个坐标的分值 返回最大分值 --- 进攻
      //在winlistP中搜素有效坐标，并计算每个坐标的分值，返回最大的分值 --- 防守
      //同等分值下 进攻优先
      var cell = this.getPlayCell();
      this.playChess(cell);
    },
    /**
     * 根据分值获得电脑下棋的位置
     * @return {[type]} [description]
     */
    getPlayCell () {

      var maxScore = 0,
          maxScoreWin = [];//最大分值的五元组
      this.winList.forEach(function(val){

        if(val.score > maxScore){
          maxScore = val.score;
          maxScoreWin = [];
          maxScoreWin.push(val);
        }else if(val.score == maxScore){
          maxScoreWin.push(val);
        }
      })

      //获得出现频率最高的索引
      var _self = this;
      var res = {};
      maxScoreWin.forEach(function(val){

        val.indexs.forEach(function(index){
          if(_self.cells[index].type == 0){
          
            if(res.hasOwnProperty(index)){
              res[index].num ++;
            }else{
              res[index] = {index:index,num:1};
            }
          }
        })
      })

      res = Object.values(res);
      res.sort(function(obj1,obj2){

        return obj2.num - obj1.num;
      })
      debugger
      return this.cells[res[0]['index']];
    },
    //每次下完棋后重置 winlist，把依赖某个点，且改点不是本方棋子的所有集合都删除
    resetWinList () {

      var _self = this;

      this.winList.forEach(function(val){

        var cellTypes = [];//获得五元组中每个单元格的状态
        val.indexs.forEach(function(index){

          cellTypes.push(_self.cells[index].type);
        })

        var typeSum = cellTypes.sum();
        var validNum = cellTypes.removeByVal(0).length;//有效棋子的数量
        switch(typeSum){
          case 0:val.score = _self.scoreMap['blank'];break;
          case 1: val.score = _self.scoreMap['W'];break;//代表四个空一个白色
          case 2: //代表两个白色三个空或者一个黑色四个空
            if(validNum == 1){//一个黑色
              val.score = _self.scoreMap['B'];
            }else if(validNum == 2){//两个白色
              val.score = _self.scoreMap['WW'];
            }else{
              val.score = _self.scoreMap['polluted'];
            }
          break;
          case 3: //代表有三个白色或者一个白色一个黑色
            if(validNum == 3){
              val.score = _self.scoreMap['WWW'];
            }else{
              val.score = _self.scoreMap['polluted'];
            }
          break;
          case 4: //代表有四个白色或者两个黑色或者一个黑色两个白色
            if(validNum == 4){
              val.score = _self.scoreMap['WWWW'];
            }else if(validNum == 2){
              val.score = _self.scoreMap['BB'];
            }else{
              val.score = _self.scoreMap['polluted'];
            }
          break;
          case 6://三个黑色或其他
            if(validNum === 3){
              val.score = _self.scoreMap['BBB'];
            }else{
              val.score = _self.scoreMap['polluted'];
            }
          break;
          case 8://四个黑色或其他
            if(validNum == 4){
              val.score = _self.scoreMap['BBBB'];
            }else{
              val.score = _self.scoreMap['polluted'];
            }
          break;
          default: val.score = _self.scoreMap['polluted'];break;

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
