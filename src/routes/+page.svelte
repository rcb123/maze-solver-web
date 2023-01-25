<script lang="ts">
	const invalidCharMessage = 'Error, invalid character.';
	const invalidMazeMessage = 'Invalid maze.';
	const noPathMessage = 'No path could be found!';

	let fileInput: HTMLInputElement;
	let mazeInput: HTMLTextAreaElement;
	let mazeOutput: HTMLDivElement;

	function uploadMazeFile() {
		const file: File = fileInput.files[0];
		const reader: FileReader = new FileReader();
		reader.onload = (event) => {
			const maze: string[][] = (reader.result as string).split('\n').map((row) => row.split(''));
			console.log(maze);
			const rows: number = maze.length;
			const cols: number = maze[0].length;
			const result = mazeSearch(maze, rows, cols);
			if (result === 1) {
				printMaze(maze);
			} else if (result === 0) {
				displayOutput(noPathMessage);
			} else {
				displayOutput(invalidMazeMessage);
			}
		};
		reader.readAsText(file);
	}

	function processMazeInput() {
		const input = mazeInput.value;
		const maze = input.split('\n').map((row) => row.split(''));
		const rows = maze.length;
		const cols = maze[0].length;
		const result = mazeSearch(maze, rows, cols);
		if (result === 1) {
			printMaze(maze);
		} else if (result === 0) {
			displayOutput(noPathMessage);
		} else {
			displayOutput(invalidMazeMessage);
		}
	}

	function mazeSearch(maze: string[][], rows: number, cols: number): number {
		// Track number of start and finish locations
		let start: [number, number] = [0, 0];
		let s: number = 0;
		let finish: [number, number] = [0, 0];
		let f: number = 0;

		// Optimization 1: Use a single loop to check for start and finish locations
		for (let i = 0; i < rows; i++) {
			for (let j = 0; j < cols; j++) {
				if (maze[i][j] === 'S') {
					start = [i, j];
					s++;
				} else if (maze[i][j] === 'F') {
					finish = [i, j];
					f++;
				}
			}
		}

		// Maze only valid with a single start and finish location
		if (s != 1 || f != 1) {
			return -1;
		}

		// Declare directions for neighboring cells
		const directions = [
			[-1, 0],
			[0, 1],
			[1, 0],
			[0, -1]
		];

		// Create arrays for previously explored cells and predecessor array for shortest path
		const explored: boolean[] = Array(rows * cols).fill(false);
		const predecessor: Array<[number, number]> = Array(rows * cols).fill([0, 0]);

		// Start Queue at the starting position
		let queue: Array<[number, number]> = [];
		queue.push(start);

		// Keep going until finish is found or all possible locations have been visited
		while (queue.length > 0) {
			// Get current location
			const current = queue.shift();

			if (current) {
				// Check for finish
				if (current[0] === finish[0] && current[1] === finish[1]) {
					// Backtrack through predecessor array for shortest path
					backtrack(maze, rows, cols, predecessor, start, finish);
					return 1;
				}

				for (const direction of directions) {
					const nextRow = current[0] + direction[0];
					const nextCol = current[1] + direction[1];
					const idx = nextRow * cols + nextCol;
					if (
						nextRow >= 0 &&
						nextRow < rows &&
						nextCol >= 0 &&
						nextCol < cols &&
						maze[nextRow][nextCol] !== '#' &&
						!explored[idx]
					) {
						queue.push([nextRow, nextCol]);
						explored[idx] = true;
						predecessor[idx] = current;
					}
				}
			}
		}
		return 0;
	}

	function backtrack(
		maze: string[][],
		rows: number,
		cols: number,
		predecessor: [number, number][],
		start: [number, number],
		finish: [number, number]
	) {
		let current: [number, number] | undefined = finish;
		while (current[0] !== start[0] || current[1] !== start[1]) {
			const idx: number = current[0] * cols + current[1];
			current = predecessor[idx];

			if (maze[current[0]][current[1]] !== 'S') {
				maze[current[0]][current[1]] = '*';
			}
		}
	}

	function printMaze(maze: string[][]) {
		let output = '<table>';
		for (let i = 0; i < maze.length; i++) {
			output += '<tr class="text-sm">';
			for (let j = 0; j < maze[i].length; j++) {
				if (maze[i][j] === '*') {
					output += `<td class="w-3 h-5 text-center p-1 text-red-500">${maze[i][j]}</td>`;
				} else if (maze[i][j] === '.') {
					output += `<td class="w-3 h-5 text-center p-1 text-white">${maze[i][j]}</td>`;
				} else if (maze[i][j] === 'S' || maze[i][j] === 'F') {
					output += `<td class="w-3 h-5 text-center p-1 text-blue-500">${maze[i][j]}</td>`;
				} else {
					output += `<td class="w-3 h-5 text-center p-1">${maze[i][j]}</td>`;
				}
			}
			output += '</tr>';
		}
		output += '</table>';
		displayOutput(output);
	}

	function displayOutput(output: string) {
		mazeOutput.innerHTML = output;
		console.log(output);
	}
</script>

<div class="mx-[10vw] flex w-[80vw] flex-col pt-10">
	<h1 class="text-center text-3xl">Maze Solver</h1>

	<div class="mt-4">
		<div class="flex flex-row">
			<div>
				<h2 class="text-lg">Instructions</h2>
				<p>
					In order to use the maze solver, you can either upload a text file or design it in the
					text box below.
				</p>
				<p>
					The valid characters are '.', '#', 'S', and 'F'. '#' is used to represent a wall, '.'
					represents an open space, and 'S' and 'F' represent the start and finish locations
					respectively.
				</p>
				<p>When submitting a maze, make sure to remove any empty lines.</p>
			</div>
			<div class="ml-auto mr-0 text-lg">
				<h2>Example Maze</h2>
				<table>
					<tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">F</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">S</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">#</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">#</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">#</td
						></tr
					><tr class="text-sm"
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">.</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td><td
							class="h-5 w-5 p-1 text-center">#</td
						><td class="h-5 w-5 p-1 text-center">.</td><td class="h-5 w-5 p-1 text-center">.</td
						></tr
					>
				</table>
			</div>
		</div>
	</div>

	<div class="mt-10 flex flex-row">
		<!-- Option to upload text file -->
		<form>
			<label for="file-input">Upload Maze File:<br /></label>
			<input
				type="file"
				id="file-input"
				accept=".txt"
				on:change={uploadMazeFile}
				bind:this={fileInput}
			/>
		</form>

		<!-- Option to create own maze in website -->
		<form id="maze-form" on:submit|preventDefault={processMazeInput} class="ml-auto mr-0">
			<textarea
				bind:this={mazeInput}
				rows="10"
				cols="20"
				class="textarea-bordered textarea text-xs"
			>...###.#....
##.#...####.
...#.#.#....
#.####.####.
#F..#..#.##.
###.#....#S.
#.#.####.##.
....#.#...#.
.####.#.#.#.
........#...</textarea>
			<br />
			<button type="submit" class="btn">Solve Maze</button>
		</form>
	</div>

	<div class="card mb-12 bg-base-100 shadow-xl">
		<div class="card-body">
			<h2 class="card-title">Output:</h2>
			<div class="mx-auto">
				<div id="maze-output" bind:this={mazeOutput} />
			</div>
		</div>
	</div>
</div>
