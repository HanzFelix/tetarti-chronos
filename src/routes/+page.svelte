<script>
	import Papa from 'papaparse';
	import { TimeGrid, Calendar } from '@event-calendar/core';

	let plugins = [TimeGrid];
	let allEvents = $state([]);
	let allSections = $state([]);
	let allInstructors = $state([]);

	let searchTerm = $state('');
	function updatefilter() {
		const lowerTerm = searchTerm.toLowerCase();
		filteredEvents = allEvents.filter((event) => {
			return (
				event.title.toLowerCase().includes(lowerTerm) ||
				(event.extendedProps.other_details &&
					event.extendedProps.other_details.toLowerCase().includes(lowerTerm))
			);
		});
	}
	let filteredEvents = $state([]);

	// Map day abbreviations to day numbers (0=Sun, 1=Mon, ..., 6=Sat)
	const dayMap = {
		M: 1,
		T: 2,
		W: 3,
		Th: 4,
		F: 5,
		S: 6,
		Su: 0
	};

	// Helper: parse time string "HH:mm" to hours and minutes
	function parseTime(timeStr) {
		const [hour, minute] = timeStr.split(':').map(Number);
		return { hour, minute };
	}

	function getSundayofCurrentWeek() {
		const sunday = new Date();
		const dayOfWeek = sunday.getDay(); // Sunday = 0, Monday = 1, ..., Saturday = 6
		sunday.setDate(sunday.getDate() - dayOfWeek);
		sunday.setHours(0, 0, 0, 0); // Optional: reset to midnight
		return sunday;
	}

	// Given a base date (e.g., semester start), find the date of the next given weekday
	function getNextDateOfWeekday(baseDate, weekday) {
		const date = new Date(baseDate);
		const diff = (7 + weekday - date.getDay()) % 7;
		date.setDate(date.getDate() + diff);
		return date;
	}

	// Handle CSV file upload
	function handleFileUpload(event) {
		const file = event.target.files[0];
		if (!file) return;

		Papa.parse(file, {
			header: true,
			skipEmptyLines: true,
			complete: (results) => {
				const baseDate = getSundayofCurrentWeek(); // You can set this to semester start date
				const parsedEvents = [];
				let parsedSections = new Set();
				let parsedInstructors = new Set();

				results.data.forEach((row) => {
					// Extract fields
					const {
						offering_no,
						course_no,
						class_type,
						room,
						days,
						time,
						instructor,
						section,
						no_of_hours,
						remark
					} = row;

					// Parse days string, e.g., "MTh" -> ['M', 'Th']
					// We assume days are concatenated without separator, so split carefully:
					// For example, split by uppercase letters or known abbreviations
					// Here, we split manually by checking 1 or 2 char day codes:
					if (days == 'TBA' || time == 'TBA') {
						return;
					}

					const days_arr = [];
					let i = 0;
					while (i < days.length) {
						if (days[i] === 'T' && days[i + 1] === 'h') {
							days_arr.push('Th');
							i += 2;
						} else {
							days_arr.push(days[i]);
							i += 1;
						}
					}

					// Parse time range
					const [startStr, endStr] = time.split('-');

					days_arr.forEach((dayAbbr) => {
						const weekday = dayMap[dayAbbr];
						if (weekday === undefined) return;

						// Get date for this weekday
						const date = getNextDateOfWeekday(baseDate, weekday);

						// Parse start and end times
						const { hour: startHour, minute: startMinute } = parseTime(startStr);
						const { hour: endHour, minute: endMinute } = parseTime(endStr);

						const startDate = new Date(date);
						startDate.setHours(startHour, startMinute, 0, 0);

						const endDate = new Date(date);
						endDate.setHours(endHour, endMinute, 0, 0);

						// Create event title
						const title = `${offering_no} (${class_type}) -- ${room} -- {${instructor}} \n${section} - ${course_no}`;

						parsedSections.add(section);
						parsedInstructors.add(instructor);
						parsedEvents.push({
							id: `${offering_no}-${dayAbbr}-${startStr}`,
							title,
							start: startDate,
							end: endDate,
							extendedProps: {
								other_details: `${instructor}-${section}S ${course_no} ${room} ${class_type} ${days}D`
							},
							styles: ['border: 1px solid white'],
							backgroundColor: remark % 100 > 0 ? 'var(--color-rose-200)' : 'var(--color-zinc-300)'
						});
					});
				});

				allEvents = parsedEvents; // Update calendar events

				allSections = Array.from(parsedSections);
				allSections.sort();

				allInstructors = Array.from(parsedInstructors);
				allInstructors.sort();
			}
		});
	}
</script>

<div class="container mx-auto mt-2 flex gap-2 *:px-2">
	<label for="search">Filter:</label>
	<input
		id="search"
		type="text"
		class="basis-full rounded-md border"
		placeholder="Search courses or professors..."
		onchange={updatefilter}
		bind:value={searchTerm}
	/>
	<input
		type="file"
		accept=".csv"
		oninput={handleFileUpload}
		class="basis-full border border-black"
	/>
	<button onclick={updatefilter}>Reload</button>
</div>
<div class="bg-gray-50">
	<Calendar
		{plugins}
		options={{
			view: 'timeGridWeek',
			events: filteredEvents,
			allDaySlot: false,
			slotMinTime: '06:00:00',
			slotMaxTime: '20:00:00',
			eventClassNames: 'bg-red-500',
			eventTextColor: 'black',
			dayHeaderFormat: { weekday: 'short' },
			hiddenDays: [0, 6],
			slotEventOverlap: false,
			slotHeight: 32,
			selectable: true,
			headerToolbar: {}
		}}
	/>
</div>
<div>
	{allSections.length}
	{#each allSections as section}
		<button
			type="button"
			aria-label={`btn-${section}`}
			onclick={() => {
				searchTerm = section + 'S';
				updatefilter();
			}}
			class="mx-2 border p-2">{section}</button
		>
	{/each}
</div>
<div>
	{allInstructors.length}
	{#each allInstructors as instructor}
		<button
			type="button"
			aria-label={`btn-${instructor}`}
			onclick={() => {
				searchTerm = instructor;
				updatefilter();
			}}
			class="mx-2 border bg-amber-200 p-2">{instructor}</button
		>
	{/each}
</div>
