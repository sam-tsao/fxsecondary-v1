<template>
  <div id="app">
    <h1>FXSECONDARY</h1>
    <div>Makeshift list showing latest 1000 offers on fxhash secondary market. 
    Please let the site load a little as it fetches data from IPFS.</div>
    <div>Made by <a href="https://twitter.com/sam___tsao" target="_blank">Sam Tsao</a>.</div>
    <table>
      <tr>
        <td><h3>Token name</h3></td>
        <td><h3>Creator   </h3></td>
        <td><h3>Token ID  </h3></td>
        <td><h3>Price     </h3></td>
        <td><h3>Timestamp </h3></td>
      </tr>
      <tr v-for="(offer, i) in offers" :key="i">
        <td>{{offer.name}}</td>
        <td>{{offer.creatorAlias || shortenAddress(offer.creatorAddress) }}</td>
        <td><a
            :href="'https://www.fxhash.xyz/objkt/' + offer.objktId"
            target="_blank"
        >{{offer.objktId}}</a></td>
        <td>{{offer.price}}</td>
        <td>{{offer.timestamp}}</td>
      </tr>
    </table>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  },
  data: ()=>({
    offers: [],
    tokenContract: "KT1KEa8z6vWXDJrVqtMrAeDVzsvxat3kHaCE",
    bcd: "https://api.better-call.dev/v1/",
    tzkt: "https://api.tzkt.io/v1/"
  }),
  mounted(){
    this.getOfferData()
  },
  methods: {
     shortenAddress: function (addr) {
      let first = addr.substring(0, 4);
      let last = addr.substring(31);
      return first + "..." + last;
    },
    getOfferData: async function() {
      let query = this.tzkt+"operations/transactions?target=KT1Xo5B7PNBAeynZPmca4bRh6LQow4og1Zb9&entrypoint=offer&status=applied&sort.desc=id&select=sender,parameter,timestamp&limit=1000"
      let response = await fetch(query);
      let data = await response.json();
        for(let i = 0; i < data.length; i++){
          let d = data[i];
        let offer = {}
        offer.objktId = d.parameter.value.objkt_id
        offer.creatorAddress = d.parameter.value.creator
        offer.createAlias = null
        offer.price = d.parameter.value.price / Math.pow(10, 6)
        offer.name = "loading..."
        offer.timestamp = d.timestamp
        offer.CID = await this.getCID(offer.objktId)
        offer.name = await this.getName(offer.CID)
        offer.creatorAlias = await this.getAlias(offer.creatorAddress)
        this.offers.push(offer)
      }
    },
    getCID: async function(id){
      let query = this.bcd + "tokens/mainnet/metadata?contract=" + this.tokenContract + "&token_id=" + id;
        let response = await fetch(query)
        let data = await response.json()
        let CID = data[0].token_info[Object.keys(data[0].token_info)[0]].substring(7);
        return CID
    },
    getName: async function(CID){
        const ipfslink = "https://gateway.ipfs.io/ipfs/" + CID
        const ipfsResponse = await fetch(ipfslink);
        const ipfsData = await ipfsResponse.json();
        return ipfsData.name
    },
    getAlias: async function(addr){
        let query = this.tzkt + "accounts/" + addr 
        let response = await fetch(query)
        let data =await response.json()
       return data.alias 
    },
    getCreatorAlias: async function(){
      await this.getObjktNames()
      for(let i = 0; i<this.offers.length;i++){
        let query = this.tzkt + "accounts/" + this.offers[i].creatorAddress
        let response = await fetch(query)
        let data =await response.json()
        this.offers[i].creatorAlias = data.alias
        console.log(this.offers[i].creatorAlias)
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Courier New', Courier, monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 20px;
}
tr:nth-child(even) {
  background: #ccc;
}

td {
  text-align: center;
  padding: 0.25em 1.5vw;
}
</style>
