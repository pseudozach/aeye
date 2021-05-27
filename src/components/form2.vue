<template>
	<div>
		<!-- form wrapper -->
		<fvl-form method="post" :data="form" url="/create">
		  <!-- Text input component -->
		  
			<fvl-input 
				:value.sync="form.question" 
				label="Question (Answer must be YES or NO)" 
				name="question" 
				style="margin: 10px;" 
				placeholder="e.g. BTC/USD price will be over $60k"
				required
				type="text"
			/>
		  <!-- Textarea component -->
		  <!-- <fvl-textarea :value.sync="form.bio" label="Bio" name="bio" /> -->

		  <!-- Radio component with options -->
		  <!-- <fvl-radio :checked.sync="form.pet" :options="{'cat': 'Cat', 'dog': 'Dog'}" label="Favorite pet" name="pet" /> -->

			<fvl-select
			  label="Select Oracle"
			  name="oracle"
			  placeholder="--- Available Automatic Oracles ---"
			  :allowEmpty="true"
			  :options="{ 'ok_24Rhws9bUiTwrLN8YBUjuTXBMvtxGAgnkRUWZv92jsJsXEFr68': 'AE / EUR', 'ak_2kivZfjJdxUMpBzLvdd4a7vnRczNrTayJJDo5VF9Mke21ixx9v' : 'Manual Account'}"
			  :selected.sync="form.oracle"
			  style="margin: 10px;"
			/>
			<fvl-input 
				:value.sync="form.oracle" 
				label="Manual Oracle Account" 
				name="manual oracle address" 
				placeholder="e.g. ak_P3fcnnJo5fnJCGQZntkNHCKv9sKTu2ABDNVYShVmPtf1KnipR"
				style="margin: 10px;"
			/>
			<!-- Gujarat / ceylon / hunan / Iowa -->

			<VueCtkDateTimePicker 
				v-model="datetime" 
				style="margin: 15px; width: auto;" 
				label="Market Resolve Date & Time"
				:min-date="mindatetime"
			/>


			<!-- <div id="statusholder" style="margin: 10px;" v-if="statustext != ''">{{statustext}}</div> -->


			<!-- <div id="txholderholder" v-if="txid != ''">Transaction ID: {{txid}}</div> -->
			<!-- <div id="stuffholderholder" v-if="stuff != ''">stuff: {{stuff}}</div> -->
		  <!-- Submit button -->
		  <!-- <fvl-submit>Validate</fvl-submit> -->

<!--     	<button
            type="button"
            class="btn-primary"
            @click="deployContract"
          >
            deploy!
      	</button> -->

  <!--     	<VueLoadingButton 
            aria-label="Get Precipitation Data from Oracle"
            class="button"
            @click.native="getpop"
            :loading="isLoading"
            :styled="true"
      	>Get Precipitation Data from Oracle
      	</VueLoadingButton> -->

<!--     	<button
            type="button"
            class="btn-primary"
            @click="getpop"
          >
            Get Precipitation Data from Oracle
      	</button> -->
      	
	      	<fvl-input :value.sync="form.paypervote" 
	      		label="Payment per vote (aetto)" 
	      		name="paypervote" 
	      		style="margin: 10px;"
	      		type="number"
	      		required
	      	/>

<!--     	<button
            type="button"
            class="btn-primary"
            @click="triggercontract"
          >
            Deposit to Contract
      	</button> -->


		  	<VueLoadingButton 
		        aria-label="Create Market"
		        class="button"
		        @click.native="createMarket"
		        :loading="isLoading"
		        :styled="true"
		  	>Create Market
		  	</VueLoadingButton>

		</fvl-form>
	</div>
</template>

<style>
	@import '~formvuelar/dist/formvuelar.css';
	.pointer-events-none {
		display: none;
	}
</style>

<script>
	import { FvlForm, FvlInput, FvlTextarea, FvlRadio, FvlSubmit, FvlSelect } from 'formvuelar'
	import VueLoadingButton from 'vue-loading-button'

	// const {Universal, Node, MemoryAccount, Crypto} = require('@aeternity/aepp-sdk');
	const { Universal: Ae, MemoryAccount, Node } = require('@aeternity/aepp-sdk')

	import { EventBus } from '../eventbus';

	import Vue from 'vue'
	import VueCtkDateTimePicker from 'vue-ctk-date-time-picker';
	import 'vue-ctk-date-time-picker/dist/vue-ctk-date-time-picker.css';
	Vue.component('VueCtkDateTimePicker', VueCtkDateTimePicker);
	import moment from 'moment';

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

  	const errorAsField = async fn => {
		try {
		  return { result: await fn }
		} catch (error) {
		  console.log(error)
		  return { error }
		}
	}

	// function to log any errors
	const logError = (error) => console.error(error);

	// function to decode errors from contract calling
	const decodeError = async (error) => {

	    // results from the node need to be decoded to be human readable
	    const decodedError = await client.contractDecodeData('string', error.returnValue).catch(logError);
	    console.error('error:', decodedError.value);
	};

export default {
	name: 'my-component',
	data() {
		return {
		    form: {
		    	question: '',
		        paypervote: '',
		        // cropregion: '',
		        // autooracle: '',
		        // manualoracle: '',
		        oracle: '',
		        // resolveTime: '',
		        // datetime: '',
		    	},
		    datetime: null,
		    mindatetime: null,
		    unixtime: null,
		    txid: '',
		    pop: '',
		    statustext: '',
		    stuff: '',
		    isLoading: false,
		    isLoadingContract: false,
		    userData: null,
		    marketcount: 0,
		    contractname: 'aepredict',
		    // dbref: 'aepredict', //testnet
		    dbref: 'aepredict_mainnet',
		    contractaddress: 'ct_fc3546zDYjvJzgkFKQCAErcShuRYPHLFwgW1o3RVd2wXS47nT',
		    client: null,
		    explorer_testnet: "https://explorer.testnet.aeternity.io/transactions/",
		    explorer_mainnet: "https://explorer.aeternity.io/transactions/",
		}
	},
    components: {
        FvlForm,
        FvlInput,
        FvlTextarea,
        FvlRadio,
        FvlSubmit,
        FvlSelect,
        VueLoadingButton,
        // Modal,
    },
    mounted: function () {
    	let thisthing = this
    	// YYYY-MM-DD hh:mm a
    	var mindate = moment().format('YYYY-MM-DD hh:mm a');
    	// console.log("mindate: ", mindate);
    	this.mindatetime = mindate;
		
		EventBus.$on('walletconnected', function(data){
			// console.log("form2 client: ", data);
			// console.log("this: ", this);
			// console.log("this.$options: ", this.$options);
			// console.log("contractname: 'aepredict ?= ", thisthing.contractname)
			thisthing.client = data;
			// this.$options.call();
				// var address = 'ct_fc3546zDYjvJzgkFKQCAErcShuRYPHLFwgW1o3RVd2wXS47nT'
				// var method = 'createMarket'
				// var returnType = 'int'
				// var args = ["web", 1000, 'ak_2kivZfjJdxUMpBzLvdd4a7vnRczNrTayJJDo5VF9Mke21ixx9v', 123, 'manual']
			 //  const result = await thisthing.client.contractCall(
			 //    address, 'sophia-address', address, method,  args
			 //  )
			 //  console.log("call result: ", result);
		});
		EventBus.$on('createmarketdone', function(data){
			// { result: 4, account: "ak_P3fcnnJo5fnJCGQZntkNHCKv9sKTu2ABDNVYShVmPtf1KnipR", txid: {…} }
			// console.log("createmarketdone: ", data);
			thisthing.isLoading = false;

			// console.log("adding to number of markets: ",  thisthing.marketcount);
			const explorerTransactionUrl = thisthing.explorer_mainnet+data.txid.hash;
			var fbobj = {marketId: data.result-1, account: data.account, question: thisthing.form.question, paypervote: thisthing.form.paypervote, oracle: thisthing.form.oracle.trim(), txid: explorerTransactionUrl, resolveTime: thisthing.datetime, unixtime: thisthing.unixtime, resolveType:"manual", yescount: 0, nocount: 0, balance:0, resolved: false, result: false,  createdAt: firebase.database.ServerValue.TIMESTAMP};
			console.log("fbobj ", fbobj);

		  	db.ref(thisthing.dbref).push(fbobj);

			this.$notify({
			  title: 'Create Market',
			  text: 'Tx broadcasted. Market created: ' + explorerTransactionUrl,
			  type: 'success',
			  duration: 10000,
			});

			thisthing.$emit('exit', true);
		});		
    },
    // created: function () {

    // },
    methods: {
    	async setupthings () {
    		let contractaddress = this.contractaddress
    		//testnet
    		// const NODE_URL = 'https://testnet.aeternity.io/';
    		const NODE_URL = 'https://mainnet.aeternity.io/';
    		const NODE_INTERNAL_URL = 'http://127.0.0.1:3113'
			const COMPILER_URL = 'https://compiler.aepps.com'
    		const node = await Node({ url: NODE_URL, internalUrl: NODE_INTERNAL_URL })
			const SDKInstance = await Ae({
			  nodes: [
			    // { name: 'testNet', instance: node },
			    { name: 'mainNet', instance: node },
			  ],
			  compilerUrl: 'COMPILER_URL',
			  accounts: [acc],
			  // the following account will be set as your active account for 
			  // calling functions, which can be changed in runtime:
			  address: yourPublicKey 
			})
			const height = await client.height()
			console.log('Current Block Height', height)
    		const contractObject = await SDKInstance.getContractInstance(CONTRACT_SOURCE, { contractaddress })

    		// const callResult = await contractObject.methods.sum(1 , 2)
    	},
    	setstuff(){
    		let thisthing = this
    		console.log("1setting stuff to yes ", this.stuff);
    		thisthing.stuff = "yes";
    		console.log("2setting stuff to yes ", this.stuff);
    		// this.$refs.Modal.close();
    		this.$emit('exit', true);
    	},
		// async call (code, address = 'ct_fc3546zDYjvJzgkFKQCAErcShuRYPHLFwgW1o3RVd2wXS47nT', method = 'createMarket', returnType = 'int', args = ["web", 1000, 'ak_2kivZfjJdxUMpBzLvdd4a7vnRczNrTayJJDo5VF9Mke21ixx9v', 123, 'manual']) {
		// 	console.log("this.client is null? ", this.client, this.contractname);
		// 	let thisthing = this
		// 	this.callResponse = await errorAsField((async () => {
		// 	  const result = await thisthing.client.contractCall(
		// 	    address, 'sophia-address', address, method,  args
		// 	  )
		// 	  console.log("call result: ", result);
		// 	  return Object.assign(
		// 	    result,
		// 	    { decodedRes: await result.decode(returnType) }
		// 	  )
		// 	})())
		// },
    	async createMarket(){
			let thisthing = this
    		// validate
    		if(thisthing.form.question == "" || thisthing.form.paypervote == "" || thisthing.form.oracle.trim() == "" || this.datetime == null || this.client == null){
    			// using options
				this.$notify({
				  title: 'Empty Fields',
				  text: 'Please fill out all required information.',
				  type: 'error',
				  duration: 10000,
				});
				return;
    		}
    		// console.log("this.datetime: ", this.datetime);
    		// let unixtime = new Date(this.datetime).getTime()/1000;
    		// console.log("unixtime: ", unixtime);
    		var momenttime = moment(this.datetime, 'YYYY-MM-DD hh:mm a');
    		// console.log("momenttime: ", momenttime, momenttime.unix());
    		let unixtime = momenttime.unix();
    		thisthing.unixtime = unixtime;
    		// thisthing.statustext = "Creating Market...";
    		thisthing.isLoading = true;
    		
    		EventBus.$emit('createmarket', ["\""+thisthing.form.question+"\"", thisthing.form.paypervote, thisthing.form.oracle.trim(), unixtime+"", "\"manual\""]);

   //  		// createMarket(question: string, paypervote: int, resolver: address, resolveTime: int, resolveType: string)
   //  		console.log("createMarket this.client is null? ", this.client);
   //  		// this.call(null, 'ct_fc3546zDYjvJzgkFKQCAErcShuRYPHLFwgW1o3RVd2wXS47nT', 'createMarket', 'int', ["web", 1000, 'ak_2kivZfjJdxUMpBzLvdd4a7vnRczNrTayJJDo5VF9Mke21ixx9v', 123, 'manual'])


		 //    // console.log("client: ", client);
			// const contractIns = await thisthing.getContractInstance(code, {contractAddress: 'ct_2qpmwFed6vmmY1RaowN342e68MFCPoNb2ZFnL4g6WtnmC2qxV1'}).catch(function(e){
			// 	console.log("contractIns err: ", e);
			// });
			// console.log("contractIns: ", contractIns);
			// const callResult = await contractIns.call('marketExists', [1]).catch(function(e) {
			// 	console.log("callResult error: ", e);
			// })
			// console.log("callResult: ",callResult);


   //  		const callResult = await this.client.contractCall('ct_2qpmwFed6vmmY1RaowN342e68MFCPoNb2ZFnL4g6WtnmC2qxV1', 'sophia-address', 'ct_fc3546zDYjvJzgkFKQCAErcShuRYPHLFwgW1o3RVd2wXS47nT', 'createMarket', {
			//     args: ["web", 1000, 'ak_2kivZfjJdxUMpBzLvdd4a7vnRczNrTayJJDo5VF9Mke21ixx9v', 123, 'manual'],
			//     // options: {amount: 200 * 15}
			// }).catch(decodeError);
			// console.log("callresult: ", callResult)



    	},
 
   //  	},
      	fetchData() {
	      db.ref(thisthing.dbref).on("value", snapshot => {
	        let data = snapshot.val();
	        console.log("number of markets: ", data.length);

	      });   
	    },
	    deployContract(){
	    	console.log("deployContract triggered!");
	  //   	const contractInstance = await SDKInstance.getContractInstance(CONTRACT_SOURCE) // contractAddress optional, only if interacting with existing contract
			// const deploymentTransaction = await contractInstance.deploy([params], options)
			EventBus.$emit('deploycontract', ["\"manual\""]);
	    },
	}
}


    // the code of your contract - watch out for correct indentations !
    var code = 
    `include "List.aes"

contract PredictionMarket =

    type o = oracle(string, string)

    record state = {
        index : int, 
        marketDatabase : map(int, market), 
        testvalue: int}

    record market = {
        id : int,
        question : string,
        balance : int,
        paypervote : int,
        resolver : address,
        yescount : int,
        nocount : int,
        resolveTime : int,
        resolveType : string,
        resolved : bool,
        result : bool,
        yesVoters : list(address),
        noVoters : list(address)
        }

    stateful entrypoint init() = 
        { index = 1,
            marketDatabase = {},
            testvalue = 42}
        
    stateful entrypoint createMarket(question: string, paypervote: int, resolver: address, resolveTime: int, resolveType: string) =
            let new_market : market = {
                id = state.index,
                question = question,
                balance = 0,
                paypervote = paypervote,
                resolver = resolver,
                yescount = 0,
                nocount = 0,
                resolveTime = resolveTime,
                resolveType = resolveType,
                resolved = false,
                result = false,
                yesVoters = [],
                noVoters = []}
                    
            put(state{marketDatabase[state.index] = new_market})
            put(state{index = (state.index + 1)})
            state.index

    payable stateful entrypoint joinMarket(id: int, side: bool) =
            require(marketExists(id), "Market does not exist")      
            require(state.marketDatabase[id].paypervote =< Call.value, "insufficient payment to join the market")
            put(state{marketDatabase[id].balance = (state.marketDatabase[id].balance + Call.value) })
            if (side)
                Call.caller :: state.marketDatabase[id].yesVoters
                put(state{marketDatabase[id].yesVoters = (Call.caller :: state.marketDatabase[id].yesVoters)})
                put(state{marketDatabase[id].yescount = (state.marketDatabase[id].yescount + 1) })
            else
                put(state{marketDatabase[id].noVoters = (Call.caller :: state.marketDatabase[id].noVoters)})
                put(state{marketDatabase[id].nocount = (state.marketDatabase[id].nocount + 1) })

    stateful entrypoint resolveMarket(id: int, result: bool) =
            require(marketExists(id), "Market does not exist")      
            require(state.marketDatabase[id].resolver == Call.caller, "caller can not resolve this market")
            put(state{marketDatabase[id].balance = (state.marketDatabase[id].balance + Call.value) })
            
            put(state{marketDatabase[id].result = result})
            put(state{marketDatabase[id].resolved = true})
            if (result)
                let amount = state.marketDatabase[id].balance / state.marketDatabase[id].yescount
                List.foreach(state.marketDatabase[id].yesVoters, (item) => payout(item, amount))
                put(state{marketDatabase[id].balance = 0})
            else
                let amount = state.marketDatabase[id].balance / state.marketDatabase[id].nocount
                List.foreach(state.marketDatabase[id].noVoters, (item) => payout(item, amount))
                put(state{marketDatabase[id].balance = 0})

    entrypoint marketExists(id: int) : bool =
        Map.member(id, state.marketDatabase)

    entrypoint getMarketQuestion(id: int) : string =
        require(marketExists(id), "Market does not exist") 
        state.marketDatabase[id].question

    entrypoint getMarketBalance(id: int) : int =
        require(marketExists(id), "Market does not exist") 
        state.marketDatabase[id].balance

    private stateful function payout(voter: address, amount: int) =
        Chain.spend(voter, amount)

    stateful entrypoint checkContractBalance(): int =
        Contract.balance

    stateful entrypoint paymeten() =
        Chain.spend(Call.caller, 10)
`

</script>

