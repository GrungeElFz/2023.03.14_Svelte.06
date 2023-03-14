<svelte:options immutable={true} />

<script>
	import Button from './Button.svelte';
	import { createEventDispatcher, afterUpdate } from 'svelte';
	import FaRegTrashAlt from 'svelte-icons/fa/FaRegTrashAlt.svelte';

	afterUpdate(() => {
		if (autoScroll) {
			listDiv.scrollTo(0, listDivScrollHeight);
		}

		autoScroll = false;
	});

	export let toDoLists = null;
	export let error = null;
	export let isLoading = false;
	export let disableAdding = false;
	export let disabledItems = [];

	let prevToDoLists = toDoLists;
	let inputText = '';
	let input, listDiv, listDivScrollHeight, autoScroll;

	const dispatch = createEventDispatcher();

	$: {
		autoScroll = toDoLists && prevToDoLists && toDoLists.length > prevToDoLists.length;
		prevToDoLists = toDoLists;
	}

	export function clearInput() {
		inputText = '';
	}

	export function focusInput() {
		input.focus();
	}

	function handleAddToDoLists() {
		const isNotCancelled = dispatch(
			'addtodo',
			{
				title: inputText
			},
			{
				cancelable: true
			}
		);

		if (isNotCancelled) {
			inputText = '';
		}
	}

	function handleRemoveToDoLists(id) {
		dispatch('removetodo', {
			id
		});
	}

	function handleToggleToDoLists(id, value) {
		dispatch('toggletodo', {
			id,
			value
		});
	}
</script>

<div class="toDoLists-wrapper">
	{#if isLoading}
		<p>Loading...</p>
	{:else if error}
		<p>{error}</p>
	{:else if toDoLists}
		<div class="toDoList" bind:this={listDiv}>
			<div bind:offsetHeight={listDivScrollHeight}>
				{#if toDoLists.length === 0}
					<p class="no-item-text">Nothing to do 🤷‍♀️</p>
				{:else}
					<ul>
						{#each toDoLists as toDoList, index (toDoList.id)}
							{@const { id, completed, title } = toDoList}
							<li>
								<slot {toDoList} {index} {handleToggleToDoLists}>
									<div class:completed>
										<label>
											<input
												disabled={disabledItems.includes(id)}
												on:input={(event) => {
													event.currentTarget.checked = completed;
													handleToggleToDoLists(id, !completed);
												}}
												type="checkbox"
												checked={completed}
											/>
											<slot name="title">{title}</slot>
										</label>
										<button
											disabled={disabledItems.includes(id)}
											class="remove-todo-button"
											aria-label="Remove toDoList: {title}"
											on:click={() => handleRemoveToDoLists(id)}
										>
											<span style:width="0.5rem" style:display="inline-block">
												<FaRegTrashAlt />
											</span>
										</button>
									</div>
								</slot>
							</li>
						{/each}
					</ul>
				{/if}
			</div>
		</div>
	{/if}

	<form class="toDoLists-form" on:submit|preventDefault={handleAddToDoLists}>
		<input
			disabled={disableAdding || !toDoLists}
			bind:this={input}
			bind:value={inputText}
			placeholder="New item"
		/>
		<Button type="submit" disabled={!inputText || disableAdding || !toDoLists}>Add</Button>
	</form>
</div>

<style lang="scss">
	.toDoLists-wrapper {
		background-color: rgb(25, 25, 25);
		border: 0.1rem solid rgb(85, 85, 85);
		border-radius: 0.5rem;
		.no-item-text {
			margin: 0;
			padding: 1rem;
			text-align: center;
		}
		.toDoList {
			max-height: 15rem;
			overflow: auto;
			ul {
				margin: 0;
				padding: 1rem;
				list-style: none;
				li > div {
					margin-bottom: 0.3rem;
					display: flex;
					align-items: center;
					background-color: rgb(20, 20, 20);
					border-radius: 0.3rem;
					padding: 0.5rem;
					position: relative;
					label {
						cursor: pointer;
						display: flex;
						align-items: baseline;
						padding-right: 1rem;
						input[type='checkbox'] {
							cursor: pointer;
							margin: 0 0.4rem 0 0;
						}
					}
					&.completed > label {
						opacity: 0.5;
						text-decoration: line-through;
					}
					.remove-todo-button {
						display: none;
						border: none;
						background: none;
						padding: 0.3rem;
						position: absolute;
						right: 0.3rem;
						cursor: pointer;
						&:disabled {
							opacity: 0.5;
							cursor: not-allowed;
						}
						:global(svg) {
							fill: rgb(100, 100, 100);
						}
					}
					&:hover {
						.remove-todo-button {
							display: block;
						}
					}
				}
			}
		}
		.toDoLists-form {
			padding: 1rem;
			background-color: rgb(20, 20, 20);
			border-radius: 0 0 0.5rem 0.5rem;
			border-top: 0.1rem solid rgb(85, 85, 85);
			display: flex;
			flex-wrap: wrap;
			input {
				flex: 1;
				color: white;
				background-color: rgb(50, 50, 50);
				border: 1px solid rgb(250, 135, 95);
				border-radius: 0.7rem;
				padding: 0.5rem;
				margin-right: 0.5rem;
			}
		}
	}
</style>
