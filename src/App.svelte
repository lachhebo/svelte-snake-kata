<script lang="ts">
	const possibleMoves = ["ArrowDown", "ArrowUp", "ArrowLeft", "ArrowRight"];
	var gamestarted = false;
	var gameloosed = false;
	var counter = 0;
	var direction = "ArrowRight";
	var history = new Array();
	var history_size = 5;
	var headPosition = {
		row: 0,
		column: 0
	}
	var speed = 700;
	history.push(headPosition);

	$:blocks = create_board();


	function create_board(){
		var i = 0;
		var j = 0;
		let board = [];
		while (i< 10){
			while (j <10){
				board.push({
					row: i, 
					column: j, 
					value: Math.random() < 0.05 ? "red" : "black"
				});
				j = j+1
			}
			i = i+1
			j = 0;
		}
		return board
	}

	function startgame() {
		gamestarted = true;
		moveSnake();
	}

	function updateDirection(event){
		if (possibleMoves.includes(event.key) ){
			direction = event.key
		}
	}

	function moveSnake() {
		if (gamestarted){
			var considered = true;
			switch (direction){
				case "ArrowLeft": 
					headPosition = {row:headPosition.row, column: headPosition.column-1};
					break;
				case "ArrowRight": 
					headPosition = {row:headPosition.row, column: headPosition.column+1};
					break;
				case "ArrowDown": 
					var newvalue = headPosition.row+1;
					headPosition = {row: newvalue, column: headPosition.column};
					break;
				case "ArrowUp": 
					var newvalue = headPosition.row-1;
					headPosition = {row:newvalue, column: headPosition.column};
					break;
				default: 
					headPosition = {row:headPosition.row, column: headPosition.column}
					considered = false;
					break;
			}
			checkPosition();
			if (!gameloosed && considered){
				updatehistory();
				handleGrowth();
				updateBoard(headPosition);
				
			}
			if (!gameloosed){
				setTimeout(moveSnake, speed);
			}
		}
	}

	function handleGrowth(){
		var {cell, ind} = getCurrentCell();

		if (cell.value == "red"){
			history_size +=1;
			counter +=1;
			blocks[ind] = {
				row: cell.row,
				column: cell.column,
				value: "black"
			}
			speed = speed*0.9;
			addRedBlock();
		}

	}

	function addRedBlock(){
		var rowGrow = Math.floor(Math.random() * 10);
		var columnGrow = Math.floor(Math.random() * 10);


		let i =0;
		let index = -1;
		blocks.forEach(block => {
			if (block.row == rowGrow && block.column == columnGrow){
				if (block.value == "green"){
					index = -1;
				}
				else {
					index = i;
				}
			}
			else {
				i +=1;
			}
		});

		if (index == -1){
			addRedBlock();
		}
		else{
			blocks[index] = {
				row: blocks[index].row,
				column: blocks[index].column,
				value: "red",
			}

			blocks = blocks;
		}

	}

	function checkPosition(){
		history.forEach( oldPostion => {
			if (headPosition.row == oldPostion.row && headPosition.column == oldPostion.column){
				gameloosed = true;
			}
		});
		if (headPosition.row < 0){
			gameloosed = true;
		}
		if (headPosition.row > 9){
			gameloosed = true;
		}
		if (headPosition.column < 0){
			gameloosed = true;
		}
		if (headPosition.column > 9){
			gameloosed = true;
		}
	}

	function updatehistory() {
		history.push(headPosition);
		if (history.length > history_size){
			var to_remove= history.shift();

			let i =0;
			let index = -1;
			blocks.forEach(block => {
				if (block.row == to_remove.row && block.column == to_remove.column){
					index = i;
				}
				else {
					i +=1;
				}
			});

			blocks[index] = {
				row:to_remove.row,
				column: to_remove.column,
				value: "black"
			}

			blocks = blocks;
		}
	}

	function updateBoard(headPosition){
		let i =0;
		let index = -1;
		blocks.forEach(block => {
			if (block.row == headPosition.row && block.column == headPosition.column){
				index = i;
			}
			else {
				i +=1;
			}
		});

		if (index == -1){
			gameloosed = true;
		}

		blocks[index] = {
			row:headPosition.row,
			column: headPosition.column,
			value: "green"
		}

		blocks = blocks;
	}

	function restartgame(){
		gameloosed = false;
		counter = 0;
		direction = "ArrowRight";
		history = new Array();
		history_size = 5;
		headPosition = {
			row: 0,
			column: 0
		}
		speed = 700;
		history.push(headPosition);

		blocks = create_board();
		moveSnake();
	}

	function getCurrentCell(){
		let i =0;
		let index = -1;
		blocks.forEach(block => {
			if (block.row == headPosition.row && block.column == headPosition.column){
				index = i;
			}
			else {
				i +=1;
			}
		});

		if (index != -1){
			return { cell: blocks[index], ind:index };
		}
	}


</script>

<main>
	<h1>SNAKE GAME</h1>

	{#if gamestarted && !gameloosed}
		<div class="wrapper" >
		{#each blocks as block}
			<div class="box" style="background-color:{block.value};grid-column:{block.column+1};grid-row:{block.row+1}"></div>
		{/each}
		</div>

		<div>
			<h3> You have {counter} { counter > 0 ? 'points' : 'point' }</h3>
		</div>
	{:else if gameloosed}
		<h3>You Lost</h3>
		<button on:click={restartgame}>REPLAY</button>
	{:else}
		<button on:click={startgame}>PLAY</button>
	{/if}

</main>

<svelte:window on:keydown={updateDirection} />

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

	.wrapper {
		background-color: white;
		display: grid;
		grid-template-columns: repeat(11, 0.1);
		grid-template-rows: repeat(11, 0.1);
		height: 300px;
		max-width: 300px;
		align-self: center;
		margin: 0 auto;
	}

	.box {
		margin: 1px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>