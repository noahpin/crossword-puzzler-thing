<script lang="ts">
	import { compileCrossword, newCrosswordController } from '$lib/crossword';
	import crosswordDefinition from '$lib/mini test.ipuz?raw';
	import jsonCrossword from '$lib/test.json';
	import '$lib/crossword.css';
	import { onMount } from 'svelte';
	import { parse, parsePuz } from '@xwordly/xword-parser';
	let gridEl: HTMLElement;
	let cluesEl: HTMLElement;

	type Clue = {
		x: number;
		y: number;
		clue: string;
		solution: string;
		answer: string;
		number?: number;
	};

	type Puzzle = {
		title: string;
		author: string;
		width: number, height: number;
		acrossClues: Clue[];
		downClues: Clue[];
		setter: any,
		source: any,
		document: any
	};
	let puzzle: Puzzle;
	function parsePuzzle(ipuz: string): Puzzle {
		const pz = parse(ipuz);
		let ipz = JSON.parse(ipuz);
		console.log(ipz);
		let w = pz.grid.width;
		let h = pz.grid.height;
		let parsed: Puzzle = {
			title: pz.title ?? 'Untitled Puzzle',
			// setter: {title: pz.title ?? 'Untitled Puzzle'},
			// source: {title:pz.author ?? 'Somebody somewhere' },
			author: pz.author ?? 'Somebody somewhere',
			width: w, height: h,
			acrossClues: [],
			downClues: [],
			document: {
    "mimetype": "application/vnd.js-crossword",
    "version": "1.0"
			}
		};
		for (let x = 0; x < w; x++) {
			for (let y = 0; y < h; y++) {
				let cell = ipz.puzzle[y][x];
				let cellContents = typeof cell == 'object' ? cell.cell : cell;
				if (cellContents != 0 && cellContents != '#') {
					let downClue = pz.clues.down.find((c) => {
						return c.number == cellContents;
					});
					let acrossClue = pz.clues.across.find((c) => {
						return c.number == cellContents;
					});
					let downSolution = '';
					let acrossSolution = '';
					let downSolutionComplete = false;
					let acrossSolutionComplete = false;
					for (let i = 0; i < Math.max(h, w); i++) {
						//march in both directions to try
						if (downClue != undefined && !downSolutionComplete && pz.grid.cells.length > y + i) {
							let c = pz.grid.cells[y + i][x];
							if (c.isBlack) {
								downSolutionComplete = true;
							} else {
								downSolution += c.solution;
							}
						}
						if (
							acrossClue != undefined &&
							!acrossSolutionComplete &&
							pz.grid.cells[y].length > x + i
						) {
							let c = pz.grid.cells[y][x + i];
							if (c.isBlack) {
								acrossSolutionComplete = true;
							} else {
								acrossSolution += c.solution;
							}
						}
					}
					console.log(downClue, acrossClue, downSolution, acrossSolution);
					if (downClue) {
						parsed.downClues.push({
							x: x + 1,
							y: y + 1,
							clue: `${downClue.number}. ${downClue.text} (${downSolution.length})`,
							solution: downSolution.toUpperCase(),
							answer: downSolution,
							number: downClue.number
						});
					}
					if (acrossClue) {
						parsed.acrossClues.push({
							x: x + 1,
							y: y + 1,
							clue: `${acrossClue.number}. ${acrossClue.text} (${acrossSolution.length})`,
							solution: acrossSolution.toUpperCase(),
							answer: acrossSolution,
							number: acrossClue.number
						});
					}
				}
			}
		}
		parsed.acrossClues.sort((a, b)=>a.number! - b.number!)
		parsed.downClues.sort((a, b)=>a.number! - b.number!)
		parsed.acrossClues = parsed.acrossClues.map(({number, ...rest})=>rest)
		parsed.downClues = parsed.downClues.map(({number, ...rest})=>rest)

		console.log(pz);
		return parsed;
	}
	onMount(async () => {
		let p = parsePuzzle(crosswordDefinition);
		console.log(p, jsonCrossword)
		let controller = newCrosswordController(p, gridEl, cluesEl);
	});
</script>

<div bind:this={gridEl} />
<div bind:this={cluesEl} />
