<script lang="ts">
	import { writable } from 'svelte/store'
	import { onMount } from 'svelte'

	type Task = {submission: string, description: string, date: Date | undefined, category: number, important: boolean; index: number}

	//i'm sorry for this horrific block of an array but i don't know how to import data yet 😝
	let defaultColors: Array<{color: string; text: string; custom: boolean}> = [
		{ color: "Salmon", text: "purple", custom: false},
		{ color: "PeachPuff", text: "purple", custom: false},
		{ color: "PapayaWhip", text: "purple", custom: false},
		{ color: "PaleGreen", text: "purple", custom: false},
		{ color: "PaleTurquoise", text: "purple", custom: false},
		{ color: "LightSkyBlue", text: "purple", custom: false},
		{ color: "Plum", text: "purple", custom: false},
		{ color: "LightPink", text: "purple", custom: false},
		{ color: "Crimson", text: "lemonchiffon", custom: false},
		{ color: "DarkOrange", text: "lemonchiffon", custom: false},
		{ color: "Gold", text: "purple", custom: false},
		{ color: "YellowGreen", text: "purple", custom: false},
		{ color: "LightSeaGreen", text: "lemonchiffon", custom: false},
		{ color: "SteelBlue", text: "lemonchiffon", custom: false},
		{ color: "MediumOrchid", text: "lemonchiffon", custom: false},
		{ color: "HotPink", text: "lemonchiffon", custom: false},
		{ color: "FireBrick", text: "lemonchiffon", custom: false},
		{ color: "Chocolate", text: "lemonchiffon", custom: false},
		{ color: "Orange", text: "lemonchiffon", custom: false},
		{ color: "SeaGreen", text: "lemonchiffon", custom: false},
		{ color: "Teal", text: "lemonchiffon", custom: false},
		{ color: "RoyalBlue", text: "lemonchiffon", custom: false},
		{ color: "RebeccaPurple", text: "lemonchiffon", custom: false},
		{ color: "MediumVioletRed", text: "lemonchiffon", custom: false},
		{ color: "GhostWhite", text: "purple", custom: false},
		{ color: "LightSteelBlue", text: "purple", custom: false},
		{ color: "SlateGray", text: "lemonchiffon", custom: false},
		{ color: "DarkSlateGray", text: "lemonchiffon", custom: false},
		{ color: "#008FC4", text: "lemonchiffon", custom: false},
	]

	let isFirstTime = writable("true");
	let months = writable<Array<Task>>([])
	let categories = writable<Array<{color: number, name: string}>>([
		{ color: 25, name: "Unsorted" },
	])
	let colors = writable<Array<{color: string; text: string; custom: boolean}>>([...defaultColors])
	let completedTasks = writable<Array<Task>>([])

	onMount(() => {
		const storedIsFirstTime = localStorage.getItem("isFirstTime");
		const storedTasks = localStorage.getItem("tasks");
		const storedCates = localStorage.getItem("categories");
		const storedColors = localStorage.getItem("colors");
		const storedComp = localStorage.getItem("completed");

	
		
		isFirstTime = writable(storedIsFirstTime ? String(storedIsFirstTime) : "true")
		months = writable(storedTasks ? JSON.parse(storedTasks) : [])
		categories = writable(storedCates ? JSON.parse(storedCates) : [{ color: 25, name: "Unsorted" }])
		colors = writable(storedColors ? JSON.parse(storedColors) : [...defaultColors]);
		completedTasks = writable(storedComp ? JSON.parse(storedComp) : []);

		for (let i = 0; i < $months.length; i++) {
			$months[i].date = new Date($months[i].date);
		}

		isFirstTime.subscribe((value) => {
			localStorage.setItem("isFirstTime", value.toString());
		})
		months.subscribe((value) => {
			localStorage.setItem("tasks", JSON.stringify(value));
		})
		categories.subscribe((value) => {
			localStorage.setItem("categories", JSON.stringify(value));
		})
		colors.subscribe((value) => {
			localStorage.setItem("colors", JSON.stringify(value));
		})
		completedTasks.subscribe((value) => {
			localStorage.setItem("completed", JSON.stringify(value));
		})

	})


	let cateNumber: number = 0
	let colorNumber: number = 4

	let colorSubmission: string = ""
	let addColorState: string = "colorPick"
	let showCustom: boolean = false

	let date: string | undefined
	let time: string | undefined


	let compButtonStates: Array<{restore: boolean, delete: boolean}> = [
		{ restore: false, delete: false }
	]




	let tasksubmission: string = ""
	let taskdescription: string = ""
	let isImportant: boolean = false

	let categorysubmission: string = ""

	let showCompleted: boolean = false

	let textDark: boolean = true

	let colorsOpen: boolean = false
	let sortOpen: boolean = false
	let cateOpen: boolean = false
	let addColsOpen: boolean = false
	let addCateOpen: boolean = false;

	let sortNumber: number = -1
	let sortSearch: string = ""

	function dateSortTasks() {
		for (let taskToAdd = 0; taskToAdd < $months.length; taskToAdd++) {
			let mostRecent: number = taskToAdd;
			for (let i = taskToAdd; i < $months.length; i++) {
				if (!$months[i].date && !$months[mostRecent].date && $months[i].index < $months[mostRecent].index) {
					mostRecent = i;
				} else if (($months[i].date < $months[mostRecent].date) || (!$months[mostRecent].date && $months[i].date)) {
					mostRecent = i;
				}
			}
			$months[taskToAdd] = replace(mostRecent, taskToAdd)
			months.update(() => $months);
		}
	}

	function replace(replacedIndex: number, newIndex: number): Task {
		let temp = $months[replacedIndex]
		$months[replacedIndex] = $months[newIndex]
		return temp
	}

	function indexSortTasks() {
		for (let taskToAdd = 0; taskToAdd < $months.length; taskToAdd++) {
			let mostRecent: number = taskToAdd;
			for (let i = taskToAdd; i < $months.length; i++) {
				if ($months[i].index < $months[mostRecent].index) {
					mostRecent = i;
				}
			}
			$months[taskToAdd] = replace(mostRecent, taskToAdd)
			months.update(() => $months);
		}
	}

	let isSortingByDate: boolean = false

	function toggleSortByDate() {
		if (isSortingByDate) {
			indexSortTasks();
		} else {
			dateSortTasks();
		}
		isSortingByDate = !isSortingByDate;
	}

	function toggleColors() {
		if (colorsOpen === false) {
			colorsOpen = true
			colorDialog.openDialog()
		} else {
			colorsOpen = false
			colorDialog.closeDialog()
			addColDialog.closeDialog()
			addColsOpen = false
		}
	}

	function toggleAddCate() {
		if (addCateOpen === false) {
			addCateOpen = true
			addCateDialog.openDialog()
		} else {
			addCateOpen = false
			addCateDialog.closeDialog()
			addColDialog.closeDialog()
			addColsOpen = false
			colorDialog.closeDialog()
			colorsOpen = false;
		}
	}

	function toggleAddColors() {
		if (!addColsOpen) {
			addColsOpen = true;
			addColDialog.openDialog()
		} else {
			addColsOpen = false;
			addColDialog.closeDialog()

		}
	}

	function toggleSort() {
		if (sortOpen === false) {
			sortOpen = true
			sortByDialog.openDialog()
		} else {
			sortOpen = false
			sortByDialog.closeDialog()
		}
	}

	function toggleOpenCate() {
		if (cateOpen === false) {
			cateOpen = true
			selectDialog.openDialog()
		} else {
			cateOpen = false
			selectDialog.closeDialog()
			addColDialog.closeDialog()
			addColsOpen = false
			colorDialog.closeDialog()
			colorsOpen = false
			addCateDialog.closeDialog()
			addCateOpen = false
		}
	}

	function toggleImportant() {
		isImportant = !isImportant;
	}

	function getDate(): Date | undefined {
		let currentDate: Date = new Date();
		let tempTime: string;
		let tempDate: string;
		let tempCurrDay: string = currentDate.getDate().toString();
		let tempCurrMonth: string = (currentDate.getMonth()+1).toString();
		if (!time && !date) return undefined;
		if (!time) {
			tempTime = "11:59:00"
		} else {
			tempTime = time.concat(":00");
		}
		if (tempCurrDay.length == 1) {
			tempCurrDay = "0".concat(tempCurrDay)
		}
		if (tempCurrMonth.length == 1) {
			tempCurrMonth = "0".concat(tempCurrMonth)
		}
		if (!date) {
			tempDate = currentDate.getFullYear().toString().concat('-', tempCurrMonth, '-', tempCurrDay);
		} else {
			tempDate = date;
		}
		return new Date(tempDate.concat('T', tempTime));
	}
	
	function addmonth() {
		if (tasksubmission.length < 1) {
			alert("Please input a title!")
		} else {

			let tempDate = getDate();
			if (tempDate && tempDate < new Date()) {alert("Please enter a valid date!")} else {
			$months = [
				...$months,
				{ submission: tasksubmission, description: taskdescription, date: tempDate, category: cateNumber, important: isImportant, index: $months.length }
			]
			date = undefined;
			time = undefined;
			showCompleted = false
			tasksubmission = ""
			taskdescription = ""
			cateNumber = 0
			isFirstTime.update(() => "false");
			isImportant = false
				if (isSortingByDate) {
					dateSortTasks();
				}
			}
		}
		months.update(() => $months);

	}


	function toggleCustom() {
		if (showCustom === false) {
			showCustom = true
		} else {
			showCustom = false
		}
	}

	function addCate() {
		if (categorysubmission.length < 1) {
			alert("Please input a title!")
		} else if (categorysubmission.length > 18) {
			alert("Too long! Please keep it under 18 characters")
		} else {
			addCateDialog.closeDialog()
			addCateOpen = false

			addColDialog.closeDialog()
			addColsOpen = false
			colorDialog.closeDialog()
			colorsOpen = false
			$categories = [
				...$categories,
				{color: colorNumber, name: categorysubmission}
			]
			categorysubmission = ""
			colorNumber = 4
			cateNumber = $categories.length-1
		}
		categories.update(() => $categories)
	}

	function toggleColorPick() {
		if (addColorState === "colorPick") {
			addColorState = "inputCSS"
		} else {
			addColorState = "colorPick"
		}
	}

	function selectCate(selected: number) {
		cateNumber = selected
		toggleOpenCate()
		console.log("sdffda")
		addColorDialog.closeDialog()
		addColsOpen = false
		colorDialog.closeDialog()
		colorsOpen = false
	}

	function sortCate(selected: number) {
		sortNumber = selected
		sortOpen = false
		sortByDialog.closeDialog()
	}

	function selectColor(selected: number) {
		colorNumber = selected
		colorsOpen = false
		colorDialog.closeDialog()
		addColDialog.closeDialog()
		addColsOpen = false
	}

	function completed(selected: number) {
		$completedTasks = [
			...$completedTasks,
			$months[selected],
		]
		compButtonStates = [
			...compButtonStates,
			{ restore: false, delete: false }
		]
		$months.splice(selected, 1)
		$months = $months
		months.update(() => $months);
		$completedTasks = $completedTasks
		completedTasks.update(() => $completedTasks)
	}

	function deleteComp(selected: number) {
		if (compButtonStates[selected].delete === true) {
			compButtonStates.splice(selected, 1)
			$completedTasks.splice(selected, 1)
			$completedTasks = $completedTasks
			completedTasks.update(() => $completedTasks)
		} else {
			compButtonStates[selected].delete = true
		}
	}

	function restoreTask(selected: number) {
		if (compButtonStates[selected].restore === true) {
			$months = [
				...$months,
				{ 
					submission: $completedTasks[selected].submission, 
					description: $completedTasks[selected].description, 
					date: $completedTasks[selected].date, 
					category: $completedTasks[selected].category,
					important: $completedTasks[selected].important,
					index: $months.length
				}
			]
			$completedTasks.splice(selected, 1)
			compButtonStates.splice(selected, 1)
			$completedTasks = $completedTasks
		} else {
			compButtonStates[selected].restore = true
		}
	}

	function cancelRestore(selected: number) {
		compButtonStates[selected].restore = false
	}

	function cancelDelete(selected: number) {
		compButtonStates[selected].delete = false
	}

	function toggleShowCom() {
		if (showCompleted === true) {
			showCompleted = false
		} else {
			showCompleted = true
			compButtonStates = compButtonStates.map(
 				obj => ({restore: false, delete: false})
			);
		}
	}

	function addColor() {
		if (colorSubmission.length < 1) {
			alert("Please input a color!")
		} else {
			if (textDark === true) {
				$colors = [...$colors, {color: colorSubmission , text: "purple", custom: true}]
			} else {
				$colors = [...$colors, {color: colorSubmission , text: "lemonchiffon", custom: true}]
			}
			addColDialog.closeDialog()
			colorSubmission = ""
			colors.update(() => $colors);
		}
	}

	function toggleTextColor() {
		if (textDark === true) {
			textDark = false
		} else {
			textDark = true
		}
	}

	import Dialog from './selectDialog.svelte'
	import ColDialog from './colorDialog.svelte'
    import AddCateDialog from './addCateDialog.svelte'
	import AddColDialog from './addColorDialog.svelte'
	import SortByDialog from './sortByDialog.svelte'
	let sortByDialog: undefined
	let addColDialog: undefined
	let addCateDialog: undefined
	let colorDialog: undefined
	let selectDialog: undefined
</script>
<svelte:head>
	<title>🧼that was a good bath🧼</title>
	<meta name="description" content="funny app!!" />
</svelte:head>
<section class="monthsubmission" style="padding: 0px">

	<!--ADD TASK-->
	<div class="pink border submit">
		<h3>
			Add a task
		</h3>
		<input bind:value={tasksubmission} type="text" placeholder="Task">
		<input bind:value={taskdescription} type="text" placeholder="Description (optional)">
		<span class="text" style="text-align: left; margin-bottom: 5px">Due: (optional)</span>
		<div class="half" style="overflow: visible; width: 100%;">
			<div style="grid-column-start: col1; grid-column-end: middle; padding-right: 2.5px">
				<input bind:value={date} type="date">
			</div>
			<div style="grid-column-start: middle; grid-column-end: end; padding-left: 2.5px">
				<input bind:value={time} type="time">
			</div>
		</div>

		<div style="display: block; width: 100%">
			<label class="text">Category:
				<button on:click={toggleOpenCate} style="margin-bottom: 10px; background-color: {$colors[$categories[cateNumber].color].color}; color: {$colors[$categories[cateNumber].color].text}">
					{$categories[cateNumber].name}
				</button>
			</label>
		</div>
		<!--CATEGORY CHOOSE-->
		<Dialog bind:this={selectDialog}>
			{#each $categories as { color, name }, i}
				<button on:click={() => selectCate(i)} style="background-color: {$colors[color].color}; margin: 5px; color: {$colors[color].text}">{name}</button>
			{/each}
			<button on:click={toggleAddCate} style="margin: 5px; background-color: transparent">＋</button>
		</Dialog>
		<div style="display: block; width: 100%; margin-bottom: 10px; overflow:visible">
			<label class="text">Important:
				<button on:click={toggleImportant}>
					{isImportant}
				</button>
			</label>
		</div>
		<button on:click={addmonth} style="display: block">
			Submit
		</button>
	</div>

	<AddCateDialog bind:this={addCateDialog}>
		<!--ADD CATEGORY-->
			<input bind:value={categorysubmission} type="text" placeholder="New Category Name">
			<div class="half" style="width: 100%; margin-bottom: 10px">
				<div style="grid-column-start: col1; grid-column-end: middle; margin-right: 5px">
					<label class="text">Color:
						<button on:click={toggleColors} style="background-color: {$colors[colorNumber].color}; color: {$colors[colorNumber].text}">
							🦐
						</button>
					</label>
				</div>
				{#if $colors.length > 29 && colorsOpen === true}
					<button on:click={toggleCustom} style="grid-column-start: middle; grid-column-end: end">
						{#if showCustom === false}
							Filter: All
						{:else}
							Filter: Custom
						{/if}
					</button>
				{/if}
			</div>

			<!--CHOOSE COLOR DIALOG-->
			<ColDialog bind:this={colorDialog}>
				{#if showCustom === false}
					{#each $colors as { color, text}, i}
						<button on:click={() => selectColor(i)} style="background-color: {color}; margin: 5px; color: {text}">
							🦐
						</button>
					{/each}
				{:else}
					{#each $colors as { color, text, custom }, i}
						{#if custom === true}
							<button on:click={() => selectColor(i)} style="background-color: {color}; margin: 5px; color: {text}">
								🦐
							</button>
						{/if}
					{/each}
				{/if}
				<button on:click={toggleAddColors} style="margin: 5px; background-color: transparent">＋</button>
			</ColDialog>
			<button on:click={addCate}>
				Submit
			</button>
	</AddCateDialog>

	<AddColDialog bind:this={addColDialog}>
		<!--ADD COLOR-->
		{#if addColorState === "colorPick"}
			<input type="color" bind:value={colorSubmission}>
			<div class="half" style="width: 100%">
				<button on:click={toggleColorPick} style="grid-column-start: col1; grid-column-end: middle; margin-right: 5px">Input hex/CSS?</button>
				<button on:click={toggleTextColor} style="grid-column-start: middle; grid-column-end: end; margin-left: 5px; background-color: {colorSubmission}">
					{#if textDark === true}
						<span style="color: purple">Light Text?</span>
					{:else}
						<span style="color: lemonchiffon">Dark Text?</span>
					{/if}
				</button>
			</div>
		{:else}
			<input type="text" bind:value={colorSubmission} placeholder="Input CSS Color/hex value (include #)">
			<div class="half" style="width: 100%; overflow: visible">
				<button on:click={toggleColorPick} style="grid-column-start: col1; grid-column-end: middle; margin-right: 5px">Pick Color?</button>
				<button on:click={toggleTextColor} style="grid-column-start: middle; grid-column-end: end; margin-left: 5px; background-color: {colorSubmission}">
					{#if textDark === true}
						<span style="color: purple">Light Text?</span>
					{:else}
						<span style="color: lemonchiffon">Dark Text?</span>
					{/if}
				</button>
			</div>
		{/if}
		<button on:click={addColor} style="display: block; margin-top: 10px;">
			Submit
		</button>
	</AddColDialog>

</section>


<section class="middle">
	<div class="middle border">
		<div class="shark">
			<div class="title"></div>
		</div>
		<div class="container">

			<button on:click={toggleShowCom} class="pink">
				{#if showCompleted === false}
					Show Completed Tasks
				{:else}
					Hide Completed Tasks
				{/if}
			</button>

		</div>
	</div>

</section>
<section class="todolist">
	<div class="todolist border" style="padding: 5px; margin-bottom: 10px">
		<h3>
			{#if showCompleted === false}
				To-Do
			{:else}
				Completed
			{/if}
		</h3>
		<div class="half">
			<div class="border" style="grid-column-start: col1; grid-column-end: middle; background: lemonchiffon; margin: 5px; padding: 10px; display: flex; align-content: center;">
				<input bind:value={sortSearch} placeholder="Search (leave empty to cancel)">
			</div>
			<div class="border" style="grid-column-start: middle; grid-column-end: end; background: lemonchiffon; margin: 5px; align-content: center; padding: 5px;">
				<label class="text">
					Filter:
						{#if sortNumber > -1}
							<button on:click={toggleSort} style="background-color: {$colors[$categories[sortNumber].color].color}; color: {$colors[$categories[sortNumber].color].text}">{$categories[sortNumber].name}</button>
						{:else}
							<button on:click={toggleSort}>None</button>
						{/if}
				</label>
			</div>
		</div>
		<SortByDialog bind:this={sortByDialog}>
			<button on:click={() => sortCate(-1)} style="margin: 5px;">
				Show All
			</button>
			{#each $categories as { color, name }, i}
				<button on:click={() => sortCate(i)} style="background-color: {$colors[color].color}; margin: 5px; color: {$colors[color].text}">{name}</button>
			{/each}
		</SortByDialog>
	</div>

<div class="todolist border" style="padding: 5px; max-height: 70vh; overflow-y: scroll">
	{#if ($isFirstTime == "true")}
		<div class="blahaj">
			<span class="message">Time to add a task! :3</span>
		</div>
	{:else if showCompleted === false}
		{#if $months.length < 1}
			<div class="yippee">
				<span class="message">all done!</span>
			</div>
		{:else if (
			($months.filter(obj => obj["category"] === sortNumber).length === 0 && sortNumber > -1) ||
			($months.filter(obj => obj["submission"].toLowerCase().includes(sortSearch.toLowerCase())).length === 0 && sortSearch.length > 0)
		)}
			<div class="dawg">
				<span class="message">
					i cannor find them
				</span>
			</div>
		{:else}
			<button on:click={toggleSortByDate}>
				{isSortingByDate ? 'Sort by creation date' : 'Sort by due date'}
			</button>
			{#each $months as { submission, description, date, category, important }, i}
				<!--TASK-->
				{#if (important)} 
					{#if 
						(sortNumber < 0 && sortSearch.length < 1) || 
						(sortNumber > -1 && sortNumber === category && sortSearch.length < 1) || 
						(submission.toLowerCase().includes(sortSearch.toLowerCase()) && sortSearch.length > 0 && sortNumber < 0) || 
						(sortNumber === category && sortNumber > -1 && submission.toLowerCase().includes(sortSearch.toLowerCase()) && sortSearch.length > 0)
					}
						<div class="task">
							<button on:click={() => completed(i)} style="grid-column-start: check; grid-column-end: task">
								✓
							</button>
							<!--CATEGORY OF TASK-->
							<div class="half" style="grid-column-start: task; grid-column-end: end; margin-left: 10px; ">
								<div class="border cateLabel" style="padding: 10px; background-color: {$colors[$categories[category].color].color}; color: {$colors[$categories[category].color].text}">
									{$categories[category].name}
								</div>

								<!--TITLE OF TASK-->
								<div class="green border important" style="grid-column-start: col2; grid-column-end: end; margin-left: 5px; padding: 5px;">
									<button class="pink" style="grid-column-start: start; grid-column-end: middle; margin-right: 5px">
										‼️
									</button>
									<div style="grid-column-start: middle; grid-column-end: end; padding: 10px">
										<span class="tasktitle">
											{submission}
										</span>
										{#if (date)}
											{#if (date > new Date())}
												<span style="color: purple">
													- Due {date}
												</span>
											{:else}
												<span style="color: salmon">
													- ⚠️ Overdue {date}
												</span>
											{/if}
										{/if}
									</div>
								</div>

								<!--DESCRIPTION OF TASK-->
								{#if description.length > 0}
									<p class="text" style="grid-column-start: col1; grid-column-end: end">
										{description}
									</p>
								{/if}
							</div>
						</div>
					{/if}
					{/if}
				{/each}
			{#each $months as { submission, description, date, category, important }, i}
			<!--TASK-->
				{#if (!important)}
					{#if 
						(sortNumber < 0 && sortSearch.length < 1) || 
						(sortNumber > -1 && sortNumber === category && sortSearch.length < 1) || 
						(submission.toLowerCase().includes(sortSearch.toLowerCase()) && sortSearch.length > 0 && sortNumber < 0) || 
						(sortNumber === category && sortNumber > -1 && submission.toLowerCase().includes(sortSearch.toLowerCase()) && sortSearch.length > 0)
					}
						<div class="task">
							<button on:click={() => completed(i)} style="grid-column-start: check; grid-column-end: task">
								✓
							</button>
							<!--CATEGORY OF TASK-->
							<div class="half" style="grid-column-start: task; grid-column-end: end; margin-left: 10px">
								<div class="border cateLabel" style="padding: 10px; background-color: {$colors[$categories[category].color].color}; color: {$colors[$categories[category].color].text}">
									{$categories[category].name}
								</div>

								<!--TITLE OF TASK-->
								<div class="green border" style="grid-column-start: col2; grid-column-end: end; margin-left: 5px; padding: 10px">
									<span class="tasktitle">
										{submission}
									</span>
									{#if (date)}
										{#if (date > new Date())}
											<span style="color: purple">
												- Due {date}
											</span>
										{:else}
											<span style="color: salmon">
												- ⚠️ Overdue {date}
											</span>
										{/if}
									{/if}
								</div>

								<!--DESCRIPTION OF TASK-->
								{#if description.length > 0}
									<p class="text" style="grid-column-start: col1; grid-column-end: end">
										{description}
									</p>
								{/if}
							</div>
						</div>
					{/if}
				{/if}
			{/each}
		{/if}
	{:else}
		{#if $completedTasks.length < 1}
			<div class="cat">
				<span class="message">
					no completed tasks here 👍
				</span>
			</div>
			{:else if (
				($completedTasks.filter(obj => obj["category"] === sortNumber).length === 0 && sortNumber > -1) ||
				($completedTasks.filter(obj => obj["submission"].toLowerCase().includes(sortSearch.toLowerCase())).length === 0 && sortSearch.length > 0)
			)}
			<div class="dawg">
				<span class="message">
					we do not have any of these items of which you speak, i'm afraid...
				</span>
			</div>
		{:else}
			{#each $completedTasks as { submission, description, date, category }, i}
				{#if 
					(sortNumber < 0 && sortSearch.length < 1) || 
					(sortNumber > -1 && sortNumber === category && sortSearch.length < 1) || 
					(submission.toLowerCase().includes(sortSearch.toLowerCase()) && sortSearch.length > 0 && sortNumber < 0) || 
					(sortNumber === category && sortNumber > -1 && submission.toLowerCase().includes(sortSearch.toLowerCase()) && sortSearch.length > 0)
				}
					<div class="task">
						<!--CATEGORY OF TASK-->
						<div class="half" style="grid-column-start: check; grid-column-end: end; margin-bottom: 5px;">
							<div class="border cateLabel" style="padding: 10px; background-color: {$colors[$categories[category].color].color}; color: {$colors[$categories[category].color].text}; margin-bottom: 2.5px">
								{$categories[category].name}
							</div>

							<!--TITLE OF TASK-->
							<div class="green border" style="padding: 10px; grid-column-start: col2; grid-column-end: end; margin-left: 5px; margin-bottom: 2.5px;">
								<span class="tasktitle">
									{submission}
								</span>
								{#if (date)}
									{#if (date > new Date())}
										<span style="color: purple">
											- Due {date}
										</span>
									{:else}
										<span style="color: salmon">
											- ⚠️ Overdue {date}
										</span>
									{/if}
								{/if}
							</div>

							<!--DESCRIPTION OF TASK-->
							{#if description.length > 0}
								<p class="text" style="grid-column-start: col1; grid-column-end: end;">
									{description}
								</p>
							{/if}
							
						</div>
							<div class="half" style="grid-column-start: check; grid-column-end: end">
								{#if compButtonStates[i].restore === false}
									<button on:click={() => restoreTask(i)} style="grid-column-start: col1; margin-right: 2.5px; grid-column-end: middle; overflow-x: hidden;">
										Restore Task
									</button>
								{:else}
									<div class="half" style="grid-column-start: col1; grid-column-end: middle; margin-right: 2.5px;">
										<button on:click={() => cancelRestore(i)} style="grid-column-start: col1; margin-right: 2.5px; grid-column-end: middle; overflow-x: hidden;">
											Cancel
										</button>
										<button on:click={() => restoreTask(i)} style="grid-column-start: middle; margin-left: 2.5px; grid-column-end: end; overflow-x: hidden;">
											Confirm
										</button>
									</div>
								{/if}
								{#if compButtonStates[i].delete === false}
									<button on:click={() => deleteComp(i)} class="salmon" style="grid-column-start: middle; margin-left: 2.5px; grid-column-end: end; overflow-x: hidden;">
										Delete Task
									</button>
								{:else}
									<div class="half" style="grid-column-start: middle; grid-column-end: end; margin-left: 2.5px;">
										<button on:click={() => cancelDelete(i)} class="salmon" style="grid-column-start: col1; margin-right: 2.5px; grid-column-end: middle; overflow-x: hidden;">
											Cancel
										</button>
										<button on:click={() => deleteComp(i)} class="salmon" style="grid-column-start: middle; margin-left: 2.5px; grid-column-end: end; overflow-x: hidden;">
											Confirm
										</button>
									</div>
								{/if}
							</div>
						</div>
					{/if}
				{/each}
			{/if}
		{/if}
	</div>
</section>
