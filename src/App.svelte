<script>

	let todos = []
	let todoInput;
	let isLoading = false;
	let enteredText = '';
	let todoEntered = false;

	$: if (enteredText !== ''){
		todoEntered = true;
	} else {
		todoEntered = false;
	}

	isLoading = true;
	let getTodos = () => fetch('https://svelte-todos-84c72-default-rtdb.firebaseio.com/todos.json')
	.then(res => {
		if (!res.ok) {
			throw new Error('Something went wrong!')
		}
		return res.json();
	})
	.then(data => {
		isLoading = false;
		const transformedTodos = [];
		for (const key in data) {
			transformedTodos.push({
				id: key,
				text: data[key]
			});
		}
		todos = transformedTodos;
		return todos;
	})
	.catch(err => {
		isLoading = false;
		console.log(err);
	})

	let promise = getTodos();

	function addTodo() {
		isLoading = true;
		fetch('https://svelte-todos-84c72-default-rtdb.firebaseio.com/todos.json', {
			method: 'POST',
			body: JSON.stringify(todoInput.value),
			headers: {
				'Content-Type': 'application/json'
			}
		}).then(res => {
			isLoading = false;
			if (!res.ok) {
				throw new Error('Something went wrong!')
			}
			enteredText = '';
			promise = getTodos();
			document.getElementById('todo').value = '';
		}).catch(err => {
			isLoading = false;
			console.log(err);
		})
	}

	function deleteTodo(id) {
		isLoading = true;
		fetch(`https://svelte-todos-84c72-default-rtdb.firebaseio.com/todos/${id}.json`, {
			method: 'DELETE'
		}).then(res => {
			isLoading = false;
			if (!res.ok) {
				throw new Error('Something went wrong!')
			}
			promise = getTodos();
		}).catch(err => {
			isLoading = false;
			console.log(err);
		})
	}

</script>

<style>
	h1, h2 {
		color: #FE0000;
		font-style: italic;
		margin-bottom: 0;
	}

	h1 {
		font-size: 90px;
		margin-top: 30px;
	}

	h2 {
		font-size: 40px;
		margin-top: 0;
	}

	p {
		margin-top: 15px;
		margin-bottom: 0;
		font-size: 2.5rem;
	}

	.submitBtn {
		width: 120px;
	}
</style>

<h1>GEDDITDUN</h1>
<h2>TO-DOs</h2>

<input type="text" id="todo" bind:value="{enteredText}" bind:this={todoInput}>
<button 
	class="submitBtn" 
	disabled={!todoEntered} 
	on:click={addTodo}>
	{!todoEntered ? 'Enter To-Do' : 'Add It!'}
</button>

{#await promise}
	<p>Loading...</p>
{:then todoData}
	{#each todoData as todo}
		<div>
			<p>{todo.text}</p>
			<button on:click={() => deleteTodo(todo.id)}>DONE</button>
		</div>
	{/each}
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}
