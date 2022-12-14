<script lang="ts">
	import Fa from 'svelte-fa';
	import { faImage, faTrash, faCode, faFileExport } from '@fortawesome/free-solid-svg-icons';
	import { browser } from '$app/environment';

	import type { CardData } from '../models/cardData';
	import { cardData, setCardData, resetCardData } from '../stores/cardData';
	import { jsonDownload, svgDownload, svgDownloadAsPng } from '../scripts/cardDownload';

	import Card from '../components/card.svelte';
	import Input from '../components/cardCreator/input.svelte';
	import HorInput from '../components/cardCreator/horInput.svelte';

	import ImgButton from '../components/cardCreator/imgButton.svelte';
	import Move from '../components/cardCreator/move.svelte';

	let cardElement: HTMLElement;
	let canvasElement: HTMLCanvasElement;
	let submitType: string;

	function setLocalStorage() {
		if (!browser) return;

		localStorage.setItem('card', JSON.stringify($cardData));
	}

	function readLocalStorage() {
		if (!browser) return;

		const readCard = localStorage.getItem('card');
		if (readCard) {
			const card: CardData = JSON.parse(readCard);
			if (card) {
				setCardData(card);
			}
		}
	}

	function resetCard() {
		if (!browser) return;

		resetCardData();

		location.reload();

		cardElement.scrollTop = 0;
	}

	function handleSubmit() {
		if (submitType === 'exportJson') {
			const data = localStorage.getItem('card');
			if (!data) return;

			jsonDownload(data);
		} else if (submitType === 'downloadSvg') {
			const svg = document.getElementById('svgPreview');
			if (!svg) return;

			svgDownload(svg);
		} else if (submitType === 'downloadPng') {
			const svg = document.getElementById('svgPreview');
			if (!svg) return;

			svgDownloadAsPng(svg, canvasElement);
		}
	}

	readLocalStorage();

	$: $cardData, setLocalStorage();
</script>

<Card>
	<h1 class="text-5xl p-6 font-semibold w-full text-center bg-gray-400/20">Card Creator</h1>
	<main class="creator w-full overflow-y-auto p-4" bind:this={cardElement}>
		<form class="flex flex-col" on:submit|preventDefault={handleSubmit}>
			<div class="flex justify-between items-center">
				<Input label="Name" defaultText="Ruud Verwaal" bind:value={$cardData.name} />
				<div class="flex justify-between w-3/5">
					<Input label="Primary Type" inputType="select" bind:type={$cardData.primaryType} />
					<Input label="Secondary Type" inputType="select" bind:type={$cardData.secondaryType} />
				</div>
			</div>
			<ImgButton bind:imgUrl={$cardData.imageUrl} />
			<div class="infogrid grid grid-cols-2 my-2">
				<div class="flex flex-col mr-auto">
					<Input label="Ability Name" defaultText="Overspannen" bind:value={$cardData.ability} />
					<Input
						label="Ability Description"
						defaultText="2x critical hit chance from leerling type moves"
						inputType="textarea"
						bind:value={$cardData.description}
					/>
					<div class="grid grid-cols-2 w-fit gap-4">
						<Move index={1} />
						<Move index={2} />
						<Move index={3} />
						<Move index={4} />
					</div>
				</div>

				<div class="flex flex-col">
					<HorInput label="Health" inputType="number" bind:value={$cardData.health} max={999} />
					<HorInput label="Attack" inputType="number" bind:value={$cardData.attack} max={999} />
					<HorInput
						label="Sp. Attack"
						inputType="number"
						bind:value={$cardData.spAttack}
						max={999}
					/>
					<HorInput label="Defense" inputType="number" bind:value={$cardData.defense} max={999} />
					<HorInput
						label="Sp. Defense"
						inputType="number"
						bind:value={$cardData.spDefense}
						max={999}
					/>
					<HorInput label="Speed" inputType="number" bind:value={$cardData.speed} max={999} />
					<HorInput
						label="Intelligence"
						inputType="number"
						bind:value={$cardData.intelligence}
						max={999}
					/>

					<div class="mt-auto">
						<button
							class="relative flex justify-center items-center p-4 text-xl rounded-md bg-yellow-400 hover:bg-yellow-500 transition my-2 w-full"
							type="submit"
							on:click={() => {
								submitType = 'exportJson';
							}}><Fa class="absolute text-2xl left-4" icon={faFileExport} />Export JSON</button
						>
						<button
							class="relative flex justify-center items-center p-4 text-xl rounded-md bg-blue-400 hover:bg-blue-500 transition my-2 w-full"
							type="submit"
							on:click={() => {
								submitType = 'downloadSvg';
							}}><Fa class="absolute text-2xl left-4" icon={faCode} />Download SVG</button
						>
						<button
							class="relative flex justify-center items-center p-4 text-xl rounded-md bg-green-400 hover:bg-green-500 transition my-2 w-full"
							type="submit"
							on:click={() => {
								submitType = 'downloadPng';
							}}><Fa class="absolute text-2xl left-4" icon={faImage} />Download PNG</button
						>
						<button
							class="relative flex justify-center items-center p-4 text-xl rounded-md bg-red-400 hover:bg-red-500 transition my-2 w-full"
							type="submit"
							on:click={() => {
								submitType = '';
								confirm('Are you sure you want to delete your card?') ? resetCard() : null;
							}}><Fa class="absolute text-2xl mr-2 left-4" icon={faTrash} />Reset Card</button
						>
					</div>
					<canvas bind:this={canvasElement} width="0" height="0" style="display: none;" />
				</div>
			</div>
		</form>
	</main>
</Card>

<style lang="scss">
	.creator {
		height: 42.6rem;

		.infogrid {
			grid-template-columns: 2fr 1fr;
		}

		/* width */
		&::-webkit-scrollbar {
			width: 0.4rem;
		}

		/* Track */
		&::-webkit-scrollbar-track {
			background: #f1f1f1;
		}

		/* Handle */
		&::-webkit-scrollbar-thumb {
			background: #999;
		}

		/* Handle on hover */
		&::-webkit-scrollbar-thumb:hover {
			background: #888;
		}
	}
</style>
