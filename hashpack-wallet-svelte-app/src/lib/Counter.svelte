<script lang="ts">
  import type { HashConnect, HashConnectTypes } from "hashconnect";
//   import type { TransferTransaction, AccountId } from "@hashgraph/sdk";
  import { onMount } from "svelte";

  $: accountId = "connect";
  $: topic = "";

  let saveData = {
    pairingString: "",
    privateKey: "",
    pairedWalletData: null,
    pairedAccounts: []
  };
  
  let appData = {
    name: "Hashconnect demo",
    description: "a demo for hashconnect",
    icon: "https://pbs.twimg.com/profile_images/1466508157702873090/4CsWIPR1_400x400.jpg",
  };

  let hashconnect: HashConnect;

  let TransferTransaction: any;
  let AccountId: any;

  onMount(async () => {
		let hc = await import('hashconnect');
		const { HashConnect } = hc;
		hashconnect = new HashConnect();

		const hashgraphSdk = await import("@hashgraph/sdk");
		TransferTransaction = hashgraphSdk.TransferTransaction;
		AccountId = hashgraphSdk.AccountId;
	});

  async function connectWallet() {
	console.log("hashconnect:", hashconnect);
    let initData = await hashconnect.init(appData);
    let privateKey = initData.privKey;
    console.log("privateKey: ", privateKey);
  
    let state: HashConnectTypes.ConnectionState = await hashconnect.connect();
    topic = state.topic;
    console.log("topic:", state.topic);
  
    saveData.pairingString = hashconnect.generatePairingString(state, "testnet", false);
	console.log("pairingString:", saveData.pairingString);
  
    hashconnect.findLocalWallets();
    hashconnect.connectToLocalWallet(saveData.pairingString);
  
    hashconnect.pairingEvent.once((pairingData: { accountIds: any[]; }) => {
      pairingData.accountIds.forEach((id: any) => {
        accountId = id;
        console.log(id);
      });
    });
  }

  async function sendHbar() {
    let to = "0.0.34008195";

    console.log("\n ==== send hbar ====\n");
    console.log("topic:", topic);
    console.log("accountId:", accountId);

    const provider = hashconnect.getProvider("testnet", topic, accountId);
    const signer = hashconnect.getSigner(provider);

    console.log("signer:", signer);

    let transaction = await new TransferTransaction()
      .addHbarTransfer(AccountId.fromString(accountId), -1)
      .addHbarTransfer(AccountId.fromString(to), 1)
      .freezeWithSigner(signer);

	  console.log(transaction);

    // TODO
    let res = await transaction.executeWithSigner(signer);
    console.log("res:", res);
    // alert("TODO: send hbar not working yet");
  }

</script>

<div class="counter">

	<button on:click={connectWallet}> {accountId} </button>

	<button on:click={sendHbar}>Send Hbar</button>

</div>

<style>
	/* .counter {
		display: flex;
		border-top: 1px solid rgba(0, 0, 0, 0.1);
		border-bottom: 1px solid rgba(0, 0, 0, 0.1);
		margin: 1rem 1rem;
	}

	.counter button {
		width: 2em;
		padding: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		border: 0;
		background-color: transparent;
		touch-action: manipulation;
		color: var(--text-color);
		font-size: 2rem;
	} */

	.counter button:hover {
		background-color: var(--secondary-color);
	}

	svg {
		width: 25%;
		height: 25%;
	}

	path {
		vector-effect: non-scaling-stroke;
		stroke-width: 2px;
		stroke: var(--text-color);
	}

	.counter-viewport {
		width: 8em;
		height: 4em;
		overflow: hidden;
		text-align: center;
		position: relative;
	}

	.counter-viewport strong {
		position: absolute;
		display: flex;
		width: 100%;
		height: 100%;
		font-weight: 400;
		color: var(--accent-color);
		font-size: 4rem;
		align-items: center;
		justify-content: center;
	}

	.counter-digits {
		position: absolute;
		width: 100%;
		height: 100%;
	}

	.hidden {
		top: -100%;
		user-select: none;
	}
</style>
