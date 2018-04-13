<template>
  <div class="pos">
    <el-row>
      <el-col :span='7' class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width: 100%">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="商品数量" width="50"></el-table-column>
              <el-table-column prop="price" label="商品金额" width="70"></el-table-column>

              <el-table-column label="操作" width="100" fixed="right">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="deleteSingleGood(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>

            <div class="totalDiv">
              <small>数量</small>：{{totalCount}} <small>总计</small>：{{totalMoney}} 元
            </div>

            <div class="div-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="deleteAll()">删除</el-button>
              <el-button type="success" @click="checkOut()">结账</el-button>
            </div>
            
          </el-tab-pane>
          <el-tab-pane label="挂单">挂单</el-tab-pane>
          <el-tab-pane label="外卖">外卖</el-tab-pane>
        </el-tabs>

      
      </el-col>
      <el-col :span="17">
        <div class="regular">
          <div class="regular-title">热门商品</div>
          <div class="regular-goods">
            <ul>
              <li v-for="good in regular" @click="addOrderList(good)">
                <span>{{good.goodsName}}</span>
                <span class="o-price">￥{{good.price}}元</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="goods-type">
          <el-tabs>
            <el-tab-pane label="汉堡">
              
              <div>

                <ul class='cookList'>
                  <li v-for="good in type0Goods" @click="addOrderList(good)">
                      <span class="foodImg"><img :src="good.goodsImg" width="100%"></span>
                      <span class="foodName">{{good.goodsName}}</span>
                      <span class="foodPrice">￥{{good.price}}元</span>
                  </li>
                </ul>

              </div>

            </el-tab-pane>
            <el-tab-pane label="小食">
              
              <div>

                <ul class='cookList'>
                  <li v-for="good in type1Goods" @click="addOrderList(good)">
                      <span class="foodImg"><img :src="good.goodsImg" width="100%"></span>
                      <span class="foodName">{{good.goodsName}}</span>
                      <span class="foodPrice">￥{{good.price}}元</span>
                  </li>
                </ul>

              </div>

            </el-tab-pane>
            <el-tab-pane label="饮料">
              
              <div>

                <ul class='cookList'>
                  <li v-for="good in type2Goods" @click="addOrderList(good)">
                      <span class="foodImg"><img :src="good.goodsImg" width="100%"></span>
                      <span class="foodName">{{good.goodsName}}</span>
                      <span class="foodPrice">￥{{good.price}}元</span>
                  </li>
                </ul>

              </div>

            </el-tab-pane>
            <el-tab-pane label="套餐">
              
              <div>

                <ul class='cookList'>
                  <li v-for="good in type3Goods" @click="addOrderList(good)">
                      <span class="foodImg"><img :src="good.goodsImg" width="100%"></span>
                      <span class="foodName">{{good.goodsName}}</span>
                      <span class="foodPrice">￥{{good.price}}元</span>
                  </li>
                </ul>

              </div>

            </el-tab-pane>
          </el-tabs>
        </div>

      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'pos',
  data() {
    return {
      tableData: [],
      regular: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0,
    }
  },
  created: function() {
    axios.get('http://jspang.com/DemoApi/oftenGoods.php')
    .then(response => {
      // console.log(response);
      this.regular = response.data;
    })
    .catch(e => {
      // console.log(e);
      alert('网络错误，禁止访问');
    })

    axios.get('http://jspang.com/DemoApi/typeGoods.php')
    .then(response => {
      // console.log(response);
      this.type0Goods = response.data[0];
      this.type1Goods = response.data[1];
      this.type2Goods = response.data[2];
      this.type3Goods = response.data[3];
    })
    .catch(e => {
      // console.log(e);
      alert('网络错误，禁止访问');
    })
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    console.log(orderHeight);
    document.getElementById('order-list').style.height = orderHeight + 'px'
  },
  methods: {
    addOrderList(good) {
      this.totalMoney = 0;
      this.totalCount = 0;
      //先判断商品是否已经存在于订单列表中
      //根据判断的值来写业务逻辑
      let isHave = false;
      for (let i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i].goodsId === good.goodsId) {
          isHave = true;
        }
      }
      if (isHave) {
        //改变商品数量+1
        let arr = this.tableData.filter(it => it.goodsId === good.goodsId)
        arr[0].count++
      } else {
        let newGoods = {goodsId: good.goodsId, goodsName: good.goodsName, price: good.price, count: 1};
        this.tableData.push(newGoods);
      }
      this.reCount();
    },
    //删除单个商品
    deleteSingleGood(good) {
      this.tableData = this.tableData.filter(it => it.goodsId !== good.goodsId)
      this.reCount();
    },
    //删除所有
    deleteAll() {
      this.tableData = [];
      this.reCount()
    },
    //结账
    checkOut() {
      if (this.totalCount !== 0) {
        this.tableData = [];
        this.reCount();
        this.$message({
          message: '结账成功',
          type: 'success',
        })
      } else {
        this.$message.error('账单为空');
      }
    },
    //重新计算数量和金额
    reCount() {
      this.totalMoney = 0;
      this.totalCount = 0;
      if (this.tableData) {
        //计算汇总金额和数量
        this.tableData.forEach(element => {
          this.totalCount += element.count;
          this.totalMoney = this.totalMoney + (element.price * element.count)
        })
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .pos-order {
    background-color: #F9FAFC;
    border-right: 1px solid tan;
  }
  .div-btn {
    margin-top: 10px;
  }
  .regular-title {
    height: 20px;
    border-bottom: 1px solid pink;
    background-color: #f9fafc;
    text-align: left;
  }
  .regular-goods ul li {
    list-style: none;
    float: left;
    border: 1px solid tan;
    padding: 10px;
    margin: 10px;
    background-color: #fff;
    cursor: pointer;
  }
  .o-price {
    color: red;
  }
  .goods-type {
    clear: both;
  }
  .cookList li{
       list-style: none;
       width:23%;
       border:1px solid #E5E9F2;
       height: auto;
       overflow: hidden;
       background-color:#fff;
       padding: 2px;
       float:left;
       margin: 2px;
      cursor: pointer;
   }
   .cookList li span{
       
        display: block;
        float:left;
   }
   .foodImg{
       width: 40%;
   }
   .foodName{
       font-size: 18px;
       padding-left: 10px;
       color:brown;
 
   }
   .foodPrice{
       font-size: 16px;
       padding-left: 10px;
       padding-top:10px;
   }
   .totalDiv {
     background-color: white;
     padding: 10px;
     border-bottom: 1px solid tan;
   }
</style>