<template>
  <div class="hello">
    <modal v-if= "!can_move" :game-score="score_board" :isShow="modalShow" @restartgame="againGame"></modal>
    <h1 class="score" >
      Score: <span>{{score_board}}</span>
    </h1>
    <h3 v-if="!can_move">GAME OVER</h3>
    <div class="board">
      <span v-for="(val,key) in initial_board " v-if="val" :key="val.id" :class="key+' value'+tile_value(val)">{{tile_value(val)}}</span>
    </div>
    <button @click="againGame" class="btn">重新开始</button>
  </div>
</template>

<script>
  /*获取弹出层*/
  import Modal from './modal.vue'


  var left = fold_order(["a","b","c","d"], ["1","2","3","4"], false);
  var right = fold_order(["a","b","c","d"], ["4","3","2","1"], false);
  var up = fold_order(["1","2","3","4"], ["a","b","c","d"], true);
  var down = fold_order( ["1","2","3","4"], ["d","c","b","a"], true);
  //      棋盘的分组方式
  function fold_order(xs, ys, reverse_keys){
    return xs.map(function(x){
      return ys.map(function(y){
        var key = [x,y];
        if(reverse_keys){
          return key.reverse().join("");
        }
        return key.join("");
      });
    });
  };
  /*棋盘数据*/
  var initial = {
    a1:null,a2:null,a3:null,a4:null,
      b1:null,b2:null,b3:null,b4:null,
      c1:null,c2:null,c3:null,c4:null,
      d1:null,d2:null,d3:null,d4:null
  };
  export default {
    name: 'hello',
    data () {
      return {
        arr:['a1','a2','a3','b1','b2','b3','c1','c2','c4'],
        msg: 'Welcome to 2048',
        initial_board:initial,
        tile_counter : 0,//棋子的数量
        directions : {
          37: left,
          38: up,
          39: right,
          40: down
        },
        modalShow :true
      }
    },
    computed:{
      score_board(){
        var that = this;
        return that.used_spaces(that.initial_board).map(function(key){
          return (that.initial_board[key].values.reduce(function(a, b) {
            return a + b;
          })) - that.initial_board[key].values[0];
        }).reduce(function(a,b){return a+b}, 0);
      },
      /*判断是否可以移动*/
      can_move(){
        var that = this;
        var new_board = [up,down,left,right].reduce(function(b, direction){
          return that.fold_board(b, direction);
        }, this.initial_board);
        return that.available_spaces(new_board).length > 0
      }
    },
    methods:{
      //test
      fn(){
        return 1111
      },
      /*获取棋子的值*/
      tile_value(tile){
        return tile ? tile.values[tile.values.length-1] : null;
      },
      /*处理单行*/
      fold_line(board, line) {
        var tiles = line.map(function(key){
          return board[key];
        }).filter(function(tile){
          return tile !== null
        });
        var new_tiles = [];
        if(tiles){
          //must loop so we can skip next if matched
          for(var i=0; i < tiles.length; i++){
            var tile = tiles[i];
            if(tile){
              var val = this.tile_value(tile);
              var next_tile = tiles[i+1];
              if(next_tile && val == this.tile_value(next_tile)){
                //skip next tile;
                i++;
                new_tiles.push({
                  id: next_tile.id, //keep id
                  values: tile.values.concat([val * 2])
                });
              }
              else{
                new_tiles.push(tile);
              }
              console.log(tile.id)
            }
          }
        }
        var new_line = {};
        line.forEach(function(key, i){
          new_line[key] = new_tiles[i] || null;
        });
        return new_line;
      },
      /*处理整个棋盘*/
      fold_board(board, lines){
        //copy reference
        var new_board = board;
        var that = this;
        lines.forEach(function(line){
          var new_line = that.fold_line(board, line);
          Object.keys(new_line).forEach(function(key){
            //mutate reference while building up board
            new_board = that.set_tile(new_board, key, new_line[key]);
          });
        });
        return new_board;

      },
      //获取所有的空的值的位置
      available_spaces(board){
        return Object.keys(board).filter(function(key){
          return board[key] === null
        })
      },
      /*获取所有存在的棋子*/
      used_spaces(board){
        return Object.keys(board).filter(function(key){
          return board[key] !== null
        })
      },
      /*新建棋子*/
      new_tile(initial) {
        return {
          id:this.tile_counter++,
          values:[initial]
        }
      },

      /*设置棋子在哪个位置*/
      set_tile(board, where, tile) {
        var new_board = {};
        Object.keys(board).forEach(function (key) {
          new_board[key] = (key == where) ? tile : board[key]
        });
        return new_board
      },
      /*判断两个棋盘是否相等*/
      same_board(board1, board2){
        return  Object.keys(board1).reduce(function(ret, key){
          return ret && board1[key] == board2[key];
        }, true);
      },
      /*判断是否更新棋盘*/
      setBoard:function(new_board){
        if(!this.same_board(this.initial_board, new_board)){
          this.initial_board = new_board ;
          return true;
        }
        return false;
      },
//----------组件内部函数------------
      keyHandler(e){
        if(this.directions[e.keyCode]
          && this.setBoard(this.fold_board(this.initial_board, this.directions[e.keyCode]))){
          setTimeout(function(){
                        this.setBoard(this.addTile(this.initial_board));
                    }.bind(this), 100);
        }
      },
      /*添加棋子*/
      addTile:function(board){
//        var location = this.available_spaces(board).sort(function() {
//          return .5 - Math.random();
//        }).pop();
        var availables = this.available_spaces(board),
            len = availables.length,
            radom = Math.floor(Math.random()*len),
            location = availables[radom];

        if(location){
          var two_or_four = Math.floor(Math.random() * 2, 0) ? 2 : 4;
          return this.set_tile(board, location, this.new_tile(two_or_four));
        }
        return board;
      },
      againGame(){
        this.initial_board = initial;
        this.initial_board = this.addTile(this.addTile(this.initial_board))
        this.modalShow = true;
      },
    },
    created:function(){
      window.addEventListener("keydown", this.keyHandler, false);
      this.initial_board = this.addTile(this.addTile(this.initial_board))
    },
    components:{
      Modal
    }
  }

</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  *{
    margin: 0;
    padding: 0;
  }
  .app{
    margin:10px;
    font-family: arial;
  }
  .board{
    display:block;
    position:relative;
    border:1px solid #ccc;
    width:215px;
    height:215px;
    padding:5px;
    margin: 0 auto;
  }
  .board span{
    font-family: arial;
    letter-spacing: -1px;
    display:block;
    width:50px;
    height:36px;
    position:absolute;
    text-align:center;
    color:white;
    font-weight:bold;
    font-size:20px;
    padding-top:14px;
    background-color:#ebe76f;
    border-radius: 5px;
    transition: all 100ms linear;
  }
  .a1, .b1, .c1, .d1{ left:5px; }
  .a2, .b2, .c2, .d2{ left:60px; }
  .a3, .b3, .c3, .d3{ left:115px; }
  .a4, .b4, .c4, .d4{ left:170px; }
  .a1, .a2, .a3, .a4{ top:5px; }
  .b1, .b2, .b3, .b4{ top:60px; }
  .c1, .c2, .c3, .c4{ top:115px; }
  .d1, .d2, .d3, .d4{ top:170px; }
  span.value2{ background-color:#ebb26f; }
  span.value4{ background-color:#ea6feb; }
  span.value8{ background-color:#eb6fa3; }
  span.value16{ background-color:#7a6feb; }
  span.value32{ background-color:#af6feb; }
  span.value64{ background-color:#6febcf; }
  span.value128{ background-color:#6fbeeb; }
  span.value256{ background-color:#afeb6f; }
  span.value512{ background-color:#7aeb6f; }
  span.value1024{ background-color:#e4eb6f; }
  .btn{
    margin-top:20px;
    width:200px;
    height:40px;
    outline: none;
    color: #ff5d75;
    border:1px solid #ccc;
    border-radius:4px;
    font-size:18px;
    font-weight:900;
    background-color: #fff;
  }
  .score{
    margin-bottom:30px;
    color:orange;
  }
</style>

