<!-- <template>
  <div style="width: 600px">
    <vue-table-dynamic 
      :params="params" 
      ref="table"
    >
    </vue-table-dynamic>
  </div>
</template> -->

<!-- compactMode -->
<!-- :fixed-header="true" -->
<template>
  <div style="margin: 20px;">
    <vue-good-table
      :columns="columns"
      :rows="rows"
      :search-options="{
        enabled: false,
        placeholder: 'Search Cashback Rates',
      }"
      :pagination-options="{
        enabled: true,
        perPage: 20}"
      @on-cell-click="onCellClick"
      :sort-options="{
        enabled: false,
      }"
      >
      <template slot="table-row" slot-scope="props">
        <span v-if="props.column.field == 'txid'">
          <!-- style="font-weight: bold; color: blue;" -->
          <a :href="props.row.txid" target="_blank">txid link</a> 
        </span>
        <span v-else-if="props.column.field == 'join'">
          <!-- style="font-weight: bold; color: blue;" -->
          <VueLoadingButton
            aria-label="Join Market"
            style="background-color: green;"
            class="button"
            label="up"
            @click.native="setThumbsUp"
            data-id="up"
            :styled="true"
            :loading="isLoadingJoin"
          ><font-awesome-icon icon="thumbs-up" /></VueLoadingButton>
          <VueLoadingButton
            aria-label="Join Market"
            style="margin-left: 10px; background-color: red;"
            class="button"
            @click.native="setThumbsDown"
            data-id="down"
            :styled="true"
            :loading="isLoadingJoin"
          ><font-awesome-icon icon="thumbs-down" /></VueLoadingButton>
        </span>
        <span v-else-if="props.column.field == 'resolve'">
          <!-- style="font-weight: bold; color: blue;" -->
          <VueLoadingButton
            aria-label="Resolve Market"
            style="background-color: green;"
            class="button"
            @click.native="setThumbsUp"
            :styled="true"
            :loading="isLoadingResolve"
          ><font-awesome-icon icon="thumbs-up" /></VueLoadingButton>
          <VueLoadingButton
            aria-label="Resolve Market"
            style="margin-left: 10px; background-color: red;"
            class="button"
            @click.native="setThumbsDown"
            :styled="true"
            :loading="isLoadingResolve"
          ><font-awesome-icon icon="thumbs-down" /></VueLoadingButton>
        </span>
        <span v-else-if="props.column.field == 'exit'">
          <!-- style="font-weight: bold; color: blue;" -->
          <VueLoadingButton
            aria-label="Exit Market"
            class="button"
            :styled="true"
            :loading="isLoadingExit"
          ><font-awesome-icon icon="times" /></VueLoadingButton>
        </span>
        <span v-else>
          {{props.formattedRow[props.column.field]}}
        </span>
      </template>
    </vue-good-table>
  </div>
</template>





<script>
// import VueTableDynamic from 'vue-table-dynamic'

// import VueGoodTablePlugin from 'vue-good-table';
// // import the styles
// import 'vue-good-table/dist/vue-good-table.css'
// Vue.use(VueGoodTablePlugin);


import 'vue-good-table/dist/vue-good-table.css'
import { VueGoodTable } from 'vue-good-table';
import VueLoadingButton from 'vue-loading-button'

import { library } from '@fortawesome/fontawesome-svg-core'
import { faPlus } from '@fortawesome/free-solid-svg-icons'
import { faCheck } from '@fortawesome/free-solid-svg-icons'
import { faTimes } from '@fortawesome/free-solid-svg-icons'
import { faThumbsUp } from '@fortawesome/free-solid-svg-icons'
import { faThumbsDown } from '@fortawesome/free-solid-svg-icons'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
library.add(faPlus)
library.add(faCheck)
library.add(faTimes)
library.add(faThumbsUp)
library.add(faThumbsDown)

import Vue from 'vue'
import VueMeta from 'vue-meta'
Vue.use(VueMeta)
// import { db } from '../main'

import { EventBus } from '../eventbus';
const BigNum = require('bn.js');

// Get a RTDB instance
import firebase from 'firebase/app'
import 'firebase/database'

var firebaseConfig = {
  apiKey: process.env.VUE_APP_fb_apikey,
  authDomain: process.env.VUE_APP_fb_authdomain,
  databaseURL: process.env.VUE_APP_fb_dburl,
  projectId: process.env.VUE_APP_fb_projectid,
  storageBucket: process.env.VUE_APP_fb_storagebucket,
  messagingSenderId: process.env.VUE_APP_fb_msgsenderid,
  appId: process.env.VUE_APP_fb_appid,
  measurementId: process.env.VUE_APP_fb_measurementid
};
  if (!firebase.apps.length) {
    // Initialize Firebase
    firebase.initializeApp(firebaseConfig);
  }

const db = firebase
  // .initializeApp({ databaseURL: 'https://MY-DATABASE.firebaseio.com' })
  .database()

export default {
  name: 'my-component',
   mounted: function () {
    let thisthing = this
    EventBus.$on('joinmarketdone', function(data){
      // { result: 4, account: "ak_P3fcnnJo5fnJCGQZntkNHCKv9sKTu2ABDNVYShVmPtf1KnipR", txid: {…} }
      // {result: decoded, account: thisthing.onAccount, txid: result, marketobj: passobj.marketobj, args: passobj.args}
      console.log("joinmarketdone: ", data, data.marketobj, data.args[1]);
      var marketobj = data.marketobj;
      thisthing.isLoadingJoin = false;

      if(data.args[1] == "true" || data.args[1] == true){
        var updateobj = {yescount: marketobj.yescount+1, nocount: marketobj.nocount, balance: parseInt(parseInt(marketobj.balance)+parseInt(marketobj.paypervote))};
      } else {
        var updateobj = {yescount: marketobj.yescount, nocount: marketobj.nocount+1, balance: parseInt(parseInt(marketobj.balance)+parseInt(marketobj.paypervote))};
      }
      const explorerTransactionUrl = "https://explorer.testnet.aeternity.io/transactions/"+data.txid.hash;
      // var updateobj = {yescount: newyescount, nocount: newnocount, balance: parseInt(marketobj.balance+marketobj.paypervote)};
      console.log("market, updateobj: ",thisthing.dbref, marketobj.merchant, updateobj);
        db.ref(thisthing.dbref+"/"+marketobj.merchant).update(updateobj);
      this.$notify({
        title: 'Joining Market',
        text: 'Tx broadcasted. Joined market: ' + explorerTransactionUrl,
        type: 'success',
        duration: 10000,
      });
    }); 

    EventBus.$on('resolvemarketdone', function(data){
      // { result: 4, account: "ak_P3fcnnJo5fnJCGQZntkNHCKv9sKTu2ABDNVYShVmPtf1KnipR", txid: {…} }
      // {result: decoded, account: thisthing.onAccount, txid: result, marketobj: passobj.marketobj, args: passobj.args}
      console.log("resolvemarketdone: ", data);
      var marketobj = data.marketobj;
      thisthing.isLoadingResolve = false;

      var updateobj = {balance: 0, resolved: true, result:data.args[1] };
      const explorerTransactionUrl = "https://explorer.testnet.aeternity.io/transactions/"+data.txid.hash;
      // var updateobj = {yescount: newyescount, nocount: newnocount, balance: parseInt(marketobj.balance+marketobj.paypervote)};
      console.log("market, updateobj: ",marketobj.merchant, updateobj);
        db.ref(thisthing.dbref+"/"+marketobj.merchant).update(updateobj);
      this.$notify({
        title: 'Resolving Market',
        text: 'Tx broadcasted. Resolved market: ' + explorerTransactionUrl,
        type: 'success',
        duration: 10000,
      });
    }); 

    EventBus.$on('resolvemarketerror', function(data){
      thisthing.isLoadingResolve = false;
      this.$notify({
        title: 'Resolve Market Error',
        text: data,
        type: 'error',
        duration: 10000,
      });
    });

  },
  methods: {
    setThumbsUp(){
      // console.log("setThumbsUp");
      this.fatype = "up";
    },
    setThumbsDown(){
      // console.log("setThumbsUp");
      this.fatype = "down";
    },
    onCellClick(params) {
      // console.log("row clicked: ", params.row)
      // console.log("column clicked: ", params.column)
      // console.log("event: ", params.event, event.target.getAttribute('data-id'), event.currentTarget.getAttribute('data-id'), event.target.dataset.id)
      // console.log("fatype: ", this.fatype)
      if(params.column.field == "join"){
        this.joinMarket(params.row);
      }
      if(params.column.field == "resolve"){
        // console.log("resolve column clicked")
        this.resolveMarket(params.row);
      }
      if(params.column.field == "exit"){
        this.exitMarket(params.row);
      }
      // params.row - row object 
      // params.column - column object
      // params.rowIndex - index of this row on the current page.
      // params.event - click event
    },
    // {
    //     "merchant": "-MYgY5u5gralxGXbxwkH",
    //     "account": "ST15RGYVK9ACFQWMFFA2TVASDVZH38B4VAV4WF6BJ",
    //     "createdAt": 1618876329677,
    //     "marketId": 2,
    //     "nocount": 0,
    //     "oracle": "asdasd",
    //     "paypervote": "1",
    //     "question": "test2",
    //     "resolveTime": "2021-04-24 04:52 pm",
    //     "resolveType": "manual",
    //     "txid": "https://explorer..co/txid/${data.txId}?chain=testnet",
    //     "unixtime": 1619308320,
    //     "yescount": 0,
    //     "vgt_id": 0,
    //     "originalIndex": 0
    // }
    async joinMarket(marketobj){
      let thisthing = this
      // validate
      if(marketobj.question == "" || marketobj.paypervote == "" || marketobj.oracle.trim() == "" || this.fatype == "nothing"){
        // using options
        this.$notify({
          title: 'Empty Fields',
          text: 'Failed to fetch market data, refresh and try again.',
          type: 'error',
          duration: 10000,
        });
        return;
      }
      // thisthing.statustext = "Creating Market...";
      thisthing.isLoadingJoin = true;
        
      // (joinMarket (marketId int) (side bool) (amount uint))
      // let sideobj = trueCV();
      let newyescount = marketobj.yescount;
      let newnocount = marketobj.nocount;
      let uservote = "false";
      if(thisthing.fatype == "down"){
        // sideobj = falseCV();
        uservote = "false";
        newnocount++;
      } else {
        newyescount++;
        uservote = "true";
      }

      var passobj = {args: [marketobj.marketId+"", uservote], paypervote: marketobj.paypervote, marketobj: marketobj};
      EventBus.$emit('joinmarket', passobj);

      },
    async resolveMarket(marketobj){
      let thisthing = this
      console.log("resolveMarket: ", JSON.stringify(marketobj));
      // let unixtime = new Date(this.datetime).getTime()/1000;
      // console.log("unixtime ", unixtime);
      // validate
      if(marketobj.question == "" || marketobj.paypervote == "" || marketobj.oracle.trim() == "" || this.fatype == "nothing"){
        // using options
      this.$notify({
        title: 'Empty Fields',
        text: 'Failed to fetch market data, refresh and try again.',
        type: 'error',
        duration: 10000,
      });
      return;
      }
      // thisthing.statustext = "Creating Market...";
      thisthing.isLoadingResolve = true;
        
      // (resolveMarket (marketId int) (result bool))
      // let sideobj = trueCV();
      // let newresult = false;

      let uservote = "false";
      if(thisthing.fatype == "down"){
        // sideobj = falseCV();
        uservote = "false";
        // newnocount++;
      } else {
        // newyescount++;
        uservote = "true";
      }
      var passobj = {args: [marketobj.marketId+"", uservote], paypervote: marketobj.paypervote, marketobj: marketobj};
      EventBus.$emit('resolvemarket', passobj);

      },
    async exitMarket(marketobj){
      let thisthing = this
      // console.log("exitMarket: ", JSON.stringify(marketobj), "useraddress ", this.userData.profile..testnet);
      // let unixtime = new Date(this.datetime).getTime()/1000;
      // console.log("unixtime ", unixtime);
      // validate
      if(marketobj.question == "" || marketobj.paypervote == "" || marketobj.oracle.trim() == "" || this.fatype == "nothing"){
        // using options
      this.$notify({
        title: 'Empty Fields',
        text: 'Failed to fetch market data, refresh and try again.',
        type: 'error',
        duration: 10000,
      });
      return;
      }
      // thisthing.statustext = "Creating Market...";
      thisthing.isLoadingResolve = true;
        
      var passobj = {args: [marketobj.marketId], paypervote: marketobj.paypervote, marketobj: marketobj};
      EventBus.$emit('exitmarket', passobj);

      },
    fetchData() {
      db.ref(this.dbref).on("value", snapshot => {
        if(snapshot.exists()){
          let data = snapshot.val();
          let messages = [];
          Object.keys(data).forEach(key => {
            // console.log("each data[key]: ", data[key]);
            var msgtopush = {merchant: key, ...data[key]};
            // console.log("msgtopush: ", msgtopush);
            messages.push(msgtopush);
            // messages.push({
            //   merchant: key,
            //   lolli: data[key].lolli,
            //   fold: data[key].fold,
            //   strike: data[key].strike,
            //   updatedAt: ''
            // });
          });
          this.rows = messages;          
        }

      });    
    },
  },
  created() {
    // let viewMessage = this;
    // const itemsRef = fire.database().ref("cbtable");
    this.fetchData()
  },
  data(){
    // console.log("data return this.rows: ", this.rows);
    // {account: thisthing.userData.profile.Address.testnet, question: thisthing.form.question, paypervote: thisthing.form.paypervote, oracle: thisthing.form.oracle.trim(), 
    return {
      isLoadingJoin: false,
      isLoadingResolve: false,
      isLoadingExit: false,
      fatype: 'nothing',
      userData: null,
      // dbref: 'aepredict',
      dbref: 'aepredict_mainnet',
      contractname: 'aepredict',
      columns: [
        // {
        //   label: 'Market Creator',
        //   field: 'account',
        // },
        {
          label: 'Question',
          field: 'question',
          // type: 'number',
          // type: 'percentage',
        },
        {
          label: 'Pay per Vote',
          field: 'paypervote',
          // type: 'number',
          // type: 'percentage',
        },
        {
          label: 'Yes #',
          field: 'yescount',
          type: 'number',
          // type: 'percentage',
        },
        {
          label: 'No #',
          field: 'nocount',
          type: 'number',
          // type: 'percentage',
        },
        {
          label: 'Oracle',
          field: 'oracle',
          // type: 'number',
          // type: 'percentage',
        },
        {
          label: 'Tx Id',
          field: 'txid',
          // type: 'number',
          // type: 'percentage',
        },
        {
          label: 'Resolve Time (PST)',
          field: 'resolveTime',
          // type: 'number',
          // type: 'percentage',
        },
        {
          label: 'Join',
          field: 'join',
          // type: 'number',
          // type: 'percentage',
        },
        {
          label: 'Resolve',
          field: 'resolve',
          // type: 'number',
          // type: 'percentage',
        },
        // {
        //   label: 'Exit',
        //   field: 'exit',
        //   // type: 'number',
        //   // type: 'percentage',
        // },
        // {
        //   label: 'Type',
        //   field: 'resolveType',
        //   // type: 'number',
        //   // type: 'percentage',
        // },
        // {
        //   label: 'Liquigate',
        //   field: 'liquigate',
        //   // type: 'number',
        //   // type: 'percentage',
        // },
        // {
        //   // 📅
        //   label: 'Created At',
        //   field: 'createdAt',
        //   type: 'date',
        //   dateInputFormat: 'T',
        //   dateOutputFormat: 'MMM do yyyy HH:mm:ss',
        // },
        // {
        //   label: 'Percent',
        //   field: 'cashback',
        //   type: 'percentage',
        // },
      ],
      rows: [],
      // mytableitems: []
      // rows: [
      //      { merchant:"Amazon", lolli:0.01, fold: 0.01, createdAt: ''},
      //   // { id:1, name:"John", age: 20, createdAt: '',score: 0.03343 },
      //   // { id:2, name:"Jane", age: 24, createdAt: '2011-10-31', score: 0.03343 },
      //   // { id:3, name:"Susan", age: 16, createdAt: '2011-10-30', score: 0.03343 },
      //   // { id:4, name:"Chris", age: 55, createdAt: '2011-10-11', score: 0.03343 },
      //   // { id:5, name:"Dan", age: 40, createdAt: '2011-10-21', score: 0.03343 },
      //   // { id:6, name:"John", age: 20, createdAt: '2011-10-31', score: 0.03343 },
      // ],
    };
  },
  components: {
    VueGoodTable,
    VueLoadingButton,
    FontAwesomeIcon,
  }
};
</script>





