<template>
  <div class="w-full p-4 flex flex-col">
    <!-- <h1 class="mb-4">Your Aepp</h1> -->

    <div class="border">
      <div class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Public Key <small>(from Wallet Aepp)</small>
        </div>
        <div v-if="addressResponse" class="p-2 w-3/4 bg-grey-lightest break-words">
          {{addressResponse | responseToString}}
        </div>
        <div v-else class="p-2 w-3/4 bg-grey-lightest break-words text-grey">
          Requesting Public Key from AE Wallet...
        </div>
      </div>
      <div v-if="balance" class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Balance
        </div>
        <div class="p-2 w-3/4 bg-grey-lightest">
          {{balance}}
        </div>
      </div>
      <!-- <div v-if="heightResponse" class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Height
        </div>
        <div class="p-2 w-3/4 bg-grey-lightest">
          {{heightResponse | responseToString}}
        </div> -->
      </div>

<!--       <template v-if="nodeInfoResponse">
        <div v-if="nodeInfoResponse.error" class="bg-green w-full flex flex-row font-mono border border-b">
          <div class="p-2 w-1/4">
            NodeInfo error
          </div>
          <div class="p-2 w-3/4 bg-grey-lightest break-words">
            {{nodeInfoResponse.error}}
          </div>
        </div>
        <div
          v-for="(value, name) in nodeInfoResponse.result"
          v-if="['url', 'name', 'nodeNetworkId', 'version'].includes(name)"
          class="bg-green w-full flex flex-row font-mono border border-b"
        >
          <div class="p-2 w-1/4 capitalize">
            {{name.replace('nodeNetworkId', 'NetworkId')}}
          </div>
          <div class="p-2 w-3/4 bg-grey-lightest">
            {{value}}
          </div>
        </div>
      </template> -->

<!--       <div v-if="compilerVersionResponse" class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Compiler version
        </div>
        <div class="p-2 w-3/4 bg-grey-lightest">
          {{compilerVersionResponse | responseToString}}
        </div>
      </div> -->

      <button
        v-if="addressResponse"
        class="w-32 rounded rounded-full bg-purple text-white py-2 px-4 pin-r mr-8 mt-4 text-xs"
        @click="disconnect"
      >Disconnect</button>
    </div>

    <!-- <h2 class="mt-4">Spend tokens</h2>

    <div class="border mt-4 rounded">
      <div class="bg-grey-lightest w-full flex flex-row font-mono">
        <div class="p-2 w-1/4">
          Recipient address
        </div>
        <div class="p-2 w-3/4 bg-white break-words">
          <input
            class="bg-black text-white border-b border-black p-2 w-full"
            v-model="spendTo"
            placeholder="ak_..."
          />
        </div>
      </div>
      <div class="bg-grey-lightest w-full flex flex-row font-mono">
        <div class="p-2 w-1/4">
          Tokens amount
        </div>
        <div class="p-2 w-3/4 bg-white break-words">
          <input
            class="bg-black text-white border-b border-black p-2 w-full"
            v-model="spendAmount"
          />
        </div>
      </div>
      <div class="bg-grey-lightest w-full flex flex-row font-mono">
        <div class="p-2 w-1/4">
          Payload
        </div>
        <div class="p-2 w-3/4 bg-white break-words">
          <input
            class="bg-black text-white border-b border-black p-2 w-full"
            v-model="spendPayload"
          />
        </div>
      </div>
      <button
        v-if="client"
        class="w-32 rounded rounded-full bg-purple text-white py-2 px-4 pin-r mr-8 mt-4 text-xs"
        @click="spend"
      >
        Spend
      </button>
    </div>

    <div v-if="spendResponse" class="border mt-4 mb-8 rounded">
      <div class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Send result
        </div>
        <div
          class="p-2 w-3/4 bg-grey-lightest break-words whitespace-pre-wrap"
        >{{ spendResponse | responseToFormattedJSON }}</div>
      </div>
    </div>

    <h2 class="mt-4">Compile Contract</h2>

    <div class="border mt-4 rounded">
      <div class="bg-grey-lightest w-full flex flex-row font-mono">
        <div class="p-2 w-1/4">
          Contract Code
        </div>
        <div class="p-2 w-3/4 bg-white">
          <textarea class="bg-black text-white border-b border-black p-2 w-full h-64" v-model="contractCode" placeholder="contact code"/>
        </div>
      </div>
      <button v-if="client" class="w-32 rounded rounded-full bg-purple text-white py-2 px-4 pin-r mr-8 mt-4 text-xs" @click="compile">
        Compile
      </button>
    </div>

    <div v-if="compileBytecodeResponse" class="border mt-4 mb-8 rounded">
      <div class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Compiled Code
        </div>
        <div class="p-2 w-3/4 bg-grey-lightest break-words">
          {{ compileBytecodeResponse | responseToString }}
        </div>
      </div>
    </div>

    <button
      v-if="compileBytecodeResponse && compileBytecodeResponse.result"
      class="w-32 rounded rounded-full bg-purple text-white py-2 px-4 pin-r mr-8 mt-4 text-xs"
      @click="deploy"
    >
      Deploy
    </button>

    <div v-if="deployResponse" class="border mt-4 mb-8 rounded">
      <div class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Deployed Contract
        </div>
        <div
          class="p-2 w-3/4 bg-grey-lightest break-words whitespace-pre-wrap"
        >{{ deployResponse | responseToFormattedJSON }}</div>
      </div>
    </div>

    <button
      v-if="deployResponse && deployResponse.result"
      class="w-32 rounded rounded-full bg-purple text-white py-2 px-4 pin-r mr-8 mt-4 text-xs"
      @click="call"
    >
      Call
    </button>

    <div v-if="callResponse" class="border mt-4 mb-8 rounded">
      <div class="bg-green w-full flex flex-row font-mono border border-b">
        <div class="p-2 w-1/4">
          Call Result
        </div>
        <div
          class="p-2 w-3/4 bg-grey-lightest break-words whitespace-pre-wrap"
        >{{ callResponse | responseToFormattedJSON }}</div>
      </div>
    </div> -->

  </div>
</template>

<script>
  import { EventBus } from '../eventbus';

  import { RpcAepp, Node } from '@aeternity/aepp-sdk/es'
  import WalletDetector from '@aeternity/aepp-sdk/es/utils/aepp-wallet-communication/wallet-detector'
  import BrowserWindowMessageConnection from '@aeternity/aepp-sdk/es/utils/aepp-wallet-communication/connection/browser-window-message'

  // Send wallet connection info to Aepp throug content script
  const TEST_NET_NODE_URL = 'https://testnet.aeternity.io'
  const MAIN_NET_NODE_INTERNAL_URL = 'https://mainnet.aeternity.io'
  const COMPILER_URL = 'https://compiler.aepps.com'

  const errorAsField = async fn => {
    try {
      return { result: await fn }
    } catch (error) {
      console.log(error)
      return { error }
    }
  }

  export default {
    data () {
      return {
        runningInFrame: window.parent !== window,
        client: null,
        addressResponse: null,
        heightResponse: null,
        compilerVersionResponse: null,
        nodeInfoResponse: null,
        spendTo: null,
        spendAmount: null,
        spendPayload: null,
        spendResponse: null,
        spendResult: null,
        spendError: null,
        balance: null,
        contractCode: `contract Identity =
      entrypoint main(x : int) = x`,
        byteCode: null,
        compileBytecodeResponse: null,
        contractInitState: [],
        deployResponse: null,
        callResponse: null,
        walletName: null,
        onAccount: null,
        accounts: []
      }
    },
    filters: {
      responseToString: response => `${response.error ? 'Error: ' : ''}${response.result || response.error}`,
      responseToFormattedJSON: response => response.error
        ? `Error: ${response.error}`
        : JSON.stringify(response.result, null, 4),
    },
    mounted: function () {
      // console.log("aesisgnin mounted");
      let thisthing = this
      EventBus.$on('createmarket', function(data){
        // [ "asd", "100", "ok_24Rhws9bUiTwrLN8YBUjuTXBMvtxGAgnkRUWZv92jsJsXEFr68", NaN, "manual" ]
        // console.log("mounted eventbus createmarket inside aesignin: ", data);
        thisthing.createMarket(data);
      });
      EventBus.$on('joinmarket', function(data){
        // [ "asd", "100", "ok_24Rhws9bUiTwrLN8YBUjuTXBMvtxGAgnkRUWZv92jsJsXEFr68", NaN, "manual" ]
        // console.log("mounted eventbus createmarket inside aesignin: ", data);
        thisthing.joinMarket(data);
      });
      EventBus.$on('resolvemarket', function(data){
        // [ "asd", "100", "ok_24Rhws9bUiTwrLN8YBUjuTXBMvtxGAgnkRUWZv92jsJsXEFr68", NaN, "manual" ]
        // console.log("mounted eventbus createmarket inside aesignin: ", data);
        thisthing.resolveMarket(data);
      });
    },
    methods: {
      async createMarket (data) {
        let thisthing = this

        // // worked!!!
        // const result = await this.client.contractCall(code, 'ct_2qpmwFed6vmmY1RaowN342e68MFCPoNb2ZFnL4g6WtnmC2qxV1', 'marketExists',  ['1']).catch(function(e){
        //   console.log("createMarket err: ", e);
        // })
        // console.log("result: ", result);

        this.callResponse = await errorAsField((async () => {
          const result = await this.client.contractCall(
            // this.contractCode, this.deployResponse.result.address, method,  args
            code, 'ct_2qpmwFed6vmmY1RaowN342e68MFCPoNb2ZFnL4g6WtnmC2qxV1', 'createMarket', data
          )
          var decoded = await result.decode('int');
          console.log("createMarketresult, decoded: ", result, decoded)
          EventBus.$emit('createmarketdone', {result: decoded, account: thisthing.onAccount, txid: result});
          return Object.assign(
            result,
            { decodedRes: await result.decode('int') }
          )
        })())

        // console.log("aesignin createMarket thisthing.client: ", this.client);
        // const contractIns = await this.getContractInstance(code, {contractAddress: 'ct_2qpmwFed6vmmY1RaowN342e68MFCPoNb2ZFnL4g6WtnmC2qxV1'}).catch(function(e){
        //   console.log("contractIns err: ", e);
        // });
        // console.log("contractIns: ", contractIns);
        // const callResult = await contractIns.call('marketExists', [1]).catch(function(e) {
        //   console.log("callResult error: ", e);
        // })
        // console.log("callResult: ",callResult);

      },
      async joinMarket (passobj) {
        var optionsobj = {amount: passobj.paypervote};
        console.log("joinMarket passobj: ", passobj, optionsobj);

        let thisthing = this
        this.callResponse = await errorAsField((async () => {
          const result = await this.client.contractCall(
            // this.contractCode, this.deployResponse.result.address, method,  args
            code, 'ct_2qpmwFed6vmmY1RaowN342e68MFCPoNb2ZFnL4g6WtnmC2qxV1', 'joinMarket', passobj.args, optionsobj
          )
          var decoded = await result.decode('int');
          console.log("joinMarketresult, decoded: ", result, decoded)

          EventBus.$emit('joinmarketdone', {result: decoded, account: thisthing.onAccount, txid: result, marketobj: passobj.marketobj, args: passobj.args});

          return Object.assign(
            result,
            { decodedRes: await result.decode('int') }
          )
        })())
      },
      // var passobj = {args: [marketobj.marketId+"", uservote], paypervote: marketobj.paypervote, marketobj: marketobj};
      async resolveMarket (passobj) {
        let thisthing = this
        this.callResponse = await errorAsField((async () => {
          const result = await this.client.contractCall(
            // this.contractCode, this.deployResponse.result.address, method,  args
            code, 'ct_2qpmwFed6vmmY1RaowN342e68MFCPoNb2ZFnL4g6WtnmC2qxV1', 'resolveMarket', passobj.args
          ).catch(function(e) {
            console.log("resolve error: ", e);
            EventBus.$emit('resolvemarketerror', e);
            return;
          });
          var decoded = await result.decode('int');
          console.log("resolveMarketresult, decoded: ", result, decoded)

          EventBus.$emit('resolvemarketdone', {result: decoded, account: thisthing.onAccount, txid: result, marketobj: passobj.marketobj, args: passobj.args});

          return Object.assign(
            result,
            { decodedRes: await result.decode('int') }
          )
        })())
      },
      async spend () {
        const onAccount = Object.keys(this.accounts.address.connected)[0]
        this.spendResponse = await errorAsField(this.client.spend(
          this.spendAmount,
          this.spendTo, {
            payload: this.spendPayload,
            // fee: 1
            // onAccount: onAccount
          }
        ));
      },
      async compile () {
        this.compileBytecodeResponse = await errorAsField(
          (await this.client.contractCompile(this.contractCode)).bytecode
        );
      },
      async deploy () {
        this.deployResponse = await errorAsField(this.client.contractDeploy(
          this.compileBytecodeResponse.result, this.contractCode, this.contractInitState
        ));
      },
      async call (code, method = 'main', returnType = 'int', args = ['5']) {
        this.callResponse = await errorAsField((async () => {
          const result = await this.client.contractCall(
            this.contractCode, this.deployResponse.result.address, method,  args
          )
          return Object.assign(
            result,
            { decodedRes: await result.decode(returnType) }
          )
        })())
      },
      resetState() {
        this.walletName = null
        this.pub = null
        this.balance = null
        this.addressResponse = null
      },
      async disconnect() {
        this.resetState()
        await this.client.disconnectWallet()
        setTimeout(() => this.scanForWallets(), 1000)
      },
      async getReverseWindow() {
        const iframe = document.createElement('iframe')
        iframe.src = prompt('Enter wallet URL', 'http://localhost:9000')
        iframe.style.display = 'none'
        document.body.appendChild(iframe)
        return iframe.contentWindow
      },
      async connectToWallet (wallet) {
        await this.client.connectToWallet(await wallet.getConnection())
        this.accounts = await this.client.subscribeAddress('subscribe', 'connected')
        this.pub = await this.client.address()
        this.onAccount = this.pub
        EventBus.$emit('walletconnected', this.client);
        this.balance = await this.client.getBalance(this.pub)
        this.walletName = this.client.rpcClient.info.name
        this.addressResponse = await errorAsField(this.client.address())
        this.heightResponse = await errorAsField(this.client.height())
        this.nodeInfoResponse = await errorAsField(this.client.getNodeInfo())
        this.compilerVersionResponse = await errorAsField(this.client.getCompilerVersion())
      },
      async scanForWallets () {
        const handleWallets = async function ({ wallets, newWallet }) {
          newWallet = newWallet || Object.values(wallets)[0]
          if (confirm(`Do you want to connect to wallet ${newWallet.name}`)) {
            this.detector.stopScan()

            await this.connectToWallet(newWallet)
          }
        }

        const scannerConnection = await BrowserWindowMessageConnection({
          connectionInfo: { id: 'spy' }
        })
        this.detector = await WalletDetector({ connection: scannerConnection })
        this.detector.scan(handleWallets.bind(this))
      }
    },
    async created () {
      // Open iframe with Wallet if run in top window
      // window !== window.parent || await this.getReverseWindow()

      this.client = await RpcAepp({
        name: 'aepredict',
        nodes: [
          { name: 'ae_uat', instance: await Node({ url: TEST_NET_NODE_URL }) },
          { name: 'ae_mainnet', instance: await Node({ url: MAIN_NET_NODE_INTERNAL_URL }) }
        ],
        compilerUrl: COMPILER_URL,
        onNetworkChange: async (params) => {
          this.client.selectNode(params.networkId)
          this.nodeInfoResponse = await errorAsField(this.client.getNodeInfo())
        },
        onAddressChange:  async (addresses) => {
          this.pub = await this.client.address()
          this.balance = await this.client.balance(this.pub).catch(e => '0')
          this.addressResponse = await errorAsField(this.client.address())
        },
        onDisconnect: () => {
          this.resetState()
          alert('Disconnected')
        }
      })
      this.height = await this.client.height()
      // Start looking for wallets
      await this.scanForWallets()
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
