<script lang="ts">
	import './app.css';
	import Lottie from './Lottie.svelte';
	import { browser } from '$app/environment';
	import { requestInvoice } from 'lnurl-pay/dist/umd/index';
	import { QRCodeImage } from 'svelte-qrcode-image';

	export let lnaddress: String;
	let amount = 0;
	let isShow = false;
	$: isClick = false;
	let lninvoice = '';
	let isLoadingInvoice = false;
	let hasBeenCopied = false;
	let isConfirm = false;

	const handleClick = () => {
		isClick = !isClick;
		setTimeout(
			() => {
				isShow = !isShow;
			},
			isShow ? 0 : 800
		);
	};

	const handleCancel = () => {
		handleClick();
		amount = 0;
		lninvoice = '';
		isLoadingInvoice = false;
		isConfirm = false;
		hasBeenCopied = false;
		isConfirm = false;
	};

	const handleConfirm = () => {
		isConfirm = true;
	};
	const handleDonate = async () => {
		if (browser) {
			try {
				isLoadingInvoice = true;
				const { invoice, params, successAction, validatePreimage } = await requestInvoice({
					lnUrlOrAddress: lnaddress,
					tokens: amount
				});
				console.log(params);

				console.log(invoice);
				lninvoice = invoice;
			} catch (e) {
				console.error(e);
			} finally {
				isLoadingInvoice = false;
			}
		}
	};
	const handleCharge = async (delay = 200) => {
		console.log('mouse down');

		if (amount < 10) {
			amount = amount + 1;
			delay = 100;
		} else if (amount < 50) {
			amount = amount + 5;
			delay = 100;
		} else if (amount < 100) {
			amount = amount + 5;
			delay = 50;
		} else if (amount < 500) {
			amount = amount + 10;
		} else if (amount < 5000) {
			amount = amount + 100;
			delay = 50;
		} else if (amount < 50000) {
			amount = amount + 1000;
		} else {
			amount = amount + 10000;
		}

		if (!isClick) {
			await setTimeout(() => {
				handleCharge(delay);
			}, delay);
		}
	};
	const copyInvoice = () => {
		if (browser) {
			let input = document.getElementById('invoice-input');
			// @ts-expect-error
			input.select();
			document.execCommand('copy');
			hasBeenCopied = true;
		}
	};
</script>

<div class="w-96 h-72 flex items-center justify-center">
	{#if isShow}
		<div class="card bg-base-100 shadow-xl h-72 w-96 z-50 absolute">
			<div class="card-body h-full">
				<h2 class="card-title">Donate to {lnaddress}</h2>
				{#if lninvoice}
					{#if isConfirm}
						<div class="flex flex-col items-center justify-center gap-2 h-full">
							<div>
								<p>Thank You!</p>
							</div>
							<div class="btn btn-success btn-square">
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="1.5"
									stroke="currentColor"
									class="w-6 h-6"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="M9 12.75L11.25 15 15 9.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
									/>
								</svg>
							</div>
						</div>
					{:else}
						<div class="flex w-full h-full gap-2 bg-base-300 p-2 rounded-lg">
							<a class="w-36 h-36" href="lightning:{lninvoice}">
								<QRCodeImage text={lninvoice} />
							</a>
							<div class="flex-col flex gap-2 h-full justify-between">
								<div class="font-bold">
									<p>Scan & Pay</p>
								</div>

								<div class="flex gap-2">
									<input
										id="invoice-input"
										type="text"
										class="input input-primary w-20"
										readonly
										bind:value={lninvoice}
									/>
									<div class="tooltip" data-tip={hasBeenCopied ? 'copied!' : 'copy'}>
										<button class="btn btn-square" on:click={copyInvoice}>
											<svg
												xmlns="http://www.w3.org/2000/svg"
												fill="none"
												viewBox="0 0 24 24"
												stroke-width="1.5"
												stroke="currentColor"
												class="w-6 h-6"
											>
												<path
													stroke-linecap="round"
													stroke-linejoin="round"
													d="M15.75 17.25v3.375c0 .621-.504 1.125-1.125 1.125h-9.75a1.125 1.125 0 01-1.125-1.125V7.875c0-.621.504-1.125 1.125-1.125H6.75a9.06 9.06 0 011.5.124m7.5 10.376h3.375c.621 0 1.125-.504 1.125-1.125V11.25c0-4.46-3.243-8.161-7.5-8.876a9.06 9.06 0 00-1.5-.124H9.375c-.621 0-1.125.504-1.125 1.125v3.5m7.5 10.375H9.375a1.125 1.125 0 01-1.125-1.125v-9.25m12 6.625v-1.875a3.375 3.375 0 00-3.375-3.375h-1.5a1.125 1.125 0 01-1.125-1.125v-1.5a3.375 3.375 0 00-3.375-3.375H9.75"
												/>
											</svg>
										</button>
									</div>
								</div>
								<div class="grid grid-cols-4 overflow-clip">
									<p class="font-bold col-span-2 truncate">Amount:</p>
									<p class="col-span-2 inline-flex gap-1 items-center">
										{amount}
										<span class="h-4 w-4">
											<svg
												class="w-4 h-4"
												stroke="currentColor"
												fill="currentColor"
												id="Layer_1"
												data-name="Layer 1"
												xmlns="http://www.w3.org/2000/svg"
												viewBox="0 0 360 360"
												><rect x="166.06" y="2.83" width="27.89" height="47.65" /><rect
													x="166.06"
													y="310.35"
													width="27.89"
													height="47.65"
												/><rect
													x="166.06"
													y="6.84"
													width="27.89"
													height="198.86"
													transform="translate(286.28 -73.74) rotate(90)"
												/><rect
													x="166.06"
													y="80.5"
													width="27.89"
													height="198.86"
													transform="translate(359.94 -0.08) rotate(90)"
												/><rect
													x="166.06"
													y="152.08"
													width="27.89"
													height="198.86"
													transform="translate(431.52 71.5) rotate(90)"
												/></svg
											>
										</span>
									</p>
								</div>
							</div>
						</div>
					{/if}
				{:else}
					<div class="flex flex-col h-full">
						<p class="inline-flex items-center gap-1">
							<span> Donate </span>
							<input
								type="text"
								class="input input-primary input-sm w-16
                            "
								bind:value={amount}
							/>
							<span class="h-4 w-4">
								<svg
									class="w-4 h-4"
									stroke="currentColor"
									fill="currentColor"
									id="Layer_1"
									data-name="Layer 1"
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 360 360"
									><rect x="166.06" y="2.83" width="27.89" height="47.65" /><rect
										x="166.06"
										y="310.35"
										width="27.89"
										height="47.65"
									/><rect
										x="166.06"
										y="6.84"
										width="27.89"
										height="198.86"
										transform="translate(286.28 -73.74) rotate(90)"
									/><rect
										x="166.06"
										y="80.5"
										width="27.89"
										height="198.86"
										transform="translate(359.94 -0.08) rotate(90)"
									/><rect
										x="166.06"
										y="152.08"
										width="27.89"
										height="198.86"
										transform="translate(431.52 71.5) rotate(90)"
									/></svg
								>
							</span>
						</p>
						<p>
							<span> to </span>
							<span class="link">
								{lnaddress}
							</span>
						</p>
					</div>
				{/if}
				<div class="card-actions justify-end items-end">
					<button class="btn btn-outline btn-sm" on:click={handleCancel}>cancel</button>
					{#if lninvoice}
						{#if isConfirm}
							<button class="btn btn-success btn-sm" on:click={handleCancel}>Ok</button>
						{:else}
							<button class="btn btn-success btn-sm" on:click={handleConfirm}>Confirm</button>
						{/if}
					{:else if isLoadingInvoice}
						<button class="btn btn-square loading btn-sm" />
					{:else}
						<button class="btn btn-primary btn-sm" on:click={handleDonate}>Donate</button>
					{/if}
				</div>
			</div>
		</div>
	{:else}
		<div class="w-32 h-32 z-10 flex items-center justify-center ">
			<div
				class="tooltip tooltip-open tooltip-secondary z-10"
				data-tip={amount ? amount + ' sats' : 'hold to donate'}
			>
				<button
					class="w-16 h-16"
					on:mouseup={handleClick}
					on:mousedown={handleCharge}
					on:touchstart={handleCharge}
					on:touchend={handleClick}
				>
					<Lottie lottiePath="/lightning.json" isAutoplay={false} isLoop={false} />
				</button>
			</div>
			<div class="w-32 h32  absolute">
				{#if !isClick}
					<Lottie lottiePath="/crackle.json" speed={0.2} isAutoplay={true} isLoop={true} />
				{/if}
			</div>
		</div>
	{/if}
</div>
