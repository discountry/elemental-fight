<script lang="ts">
	import { ethers } from 'ethers';
	import { onMount } from 'svelte';

	const ERC20_ABI = [
		{
			constant: true,
			inputs: [
				{
					name: '_owner',
					type: 'address'
				}
			],
			name: 'balanceOf',
			outputs: [
				{
					name: 'balance',
					type: 'uint256'
				}
			],
			payable: false,
			type: 'function'
		}
	];

	const tokenAddress = '0x348B9EaA3350EC3EfDB731FAc13EA1De234d2DE6';

	/**
	 * @type {ethers.providers.Web3Provider}
	 */
	let provider: ethers.providers.Web3Provider;
	let signer;
	/**
	 * @type {string | Promise<string>}
	 */
	let signerAddress = '';
	let tokenContract;
	let tokenBalance = 0;
	let accountBalance = '0';
	let isConnected = false;
	let isUnlocked = false;
	let initialized = false;
	let isPermanentlyDisconnected = false;
	let metamaskStatus = '...';

	async function connectWallet() {
		const { ethereum } = window;

		if (typeof window.ethereum !== 'undefined') {
			// connect
			await ethereum.request({ method: 'eth_requestAccounts' }).catch((err: { code: number }) => {
				if (err.code === 4001) {
					// EIP-1193 userRejectedRequest
					alert('You need to install MetaMask');
				} else {
					console.error(err);
				}
			});

			// get provider
			provider = new ethers.providers.Web3Provider(ethereum);

			// get signer
			signer = provider.getSigner();

			// get connected wallet address
			signerAddress = await signer.getAddress();

			// get account balance
			accountBalance = (await provider.getBalance(signerAddress)).toString();

			// erc-20 token
			// tokenContract = new ethers.Contract(tokenAddress, ERC20_ABI, provider);
			// tokenBalance = await tokenContract.balanceOf(signerAddress);

			// update on account change
			ethereum.on('accountsChanged', function (accounts: string[]) {
				signerAddress = accounts[0];
			});
		} else {
			alert('MetaMask has not been detected in your browser!');
			console.log('MetaMask has not been detected in your browser!');
		}
	}

	onMount(() => {
		// connectWallet();

		var accountInterval = setInterval(function () {
			if (provider) {
				({ isConnected, isUnlocked, initialized, isPermanentlyDisconnected } =
					window.ethereum._state);

				if (isConnected && !isPermanentlyDisconnected && initialized) {
					if (isUnlocked) {
						metamaskStatus = `<span style="color: green;">MetaMask has been successfully detected in your browser, with external connectivity, and is unlocked.</span>`;

						// get account balance
						provider.getBalance(signerAddress).then((_accountBalance) => {
							accountBalance = _accountBalance.toString();
						});

						// erc-20 token balance
						// tokenContract.balanceOf(signerAddress).then((_tokenBalance) => {
						// 	tokenBalance = _tokenBalance;
						// });
					} else {
						metamaskStatus = `<span style="color: red;">Please unlock your MetaMask and reload.</span>`;
					}
				} else {
					if (isUnlocked) {
						metamaskStatus = `<span style="color: red;">MetaMask has been successfully detected in your browser, but without external connectivity.</span>`;
					} else {
						metamaskStatus = `<span style="color: red;">Please unlock your MetaMask and reload.</span>`;
					}
				}
			}
		}, 3000);
	});
</script>

<main>
	<h1>MetaMask Integration</h1>
	<button on:click={connectWallet}>Connect Metamask</button>
	<p><b>Connected Account</b>: {signerAddress}</p>
	<p><b>ETH Balance</b>: {ethers.utils.formatEther(accountBalance)}</p>
	<!-- <p><b>WHITTLE Balance</b>: {ethers.utils.formatEther(tokenBalance)}</p> -->

	<br />
	<p>{@html metamaskStatus}</p>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: red;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
