<script lang="ts">
	import type {
		CalendarOptions,
		EventClickArg,
	} from '@fullcalendar/core';
	import FullCalendar, { Draggable } from 'svelte-fullcalendar';
	import interactionPlugin from '@fullcalendar/interaction';
	import resourceTimeline from '@fullcalendar/resource-timeline';

	let calendarComponentRef;

	var staff = [
		{ id: 'sam', title: 'Sam' },
		{ id: 'cha', title: 'Charlotte' },
		{ id: 'ivy', title: 'Ivy' },
	];

	let selectedEventWrapper: EventClickArg;
	let eventsFromStorage = [];

	try {
		eventsFromStorage = JSON.parse(localStorage.getItem('events'));
	} catch {}

	//console.log(JSON.stringify(eventsFromStorage));

	let options: CalendarOptions = {
		schedulerLicenseKey: 'CC-Attribution-NonCommercial-NoDerivatives',
		plugins: [resourceTimeline, interactionPlugin],
		height: '500px',
		resourceAreaWidth: '120px',
		droppable: true,
		editable: true,
		initialView: 'resourceTimelineOneDay',
		views: {
			resourceTimelineOneDay: {
				type: 'resourceTimeline',
				duration: { days: 1 },
			},
		},
		events: eventsFromStorage,
		resources: staff,
		slotMinTime: '08:00:00',
		slotMaxTime: '19:00:00',
		eventDrop: (event) => {
			persistEvents();
		},
		eventReceive: (event) => {
			persistEvents();
		},
		eventResize: () => {
			persistEvents();
		},
		eventRemove: () => {
			persistEvents();
		},
		eventLeave: (event) => {
			//console.log('Event dropped off calendar!', event.event);
		},
		eventClick: (event) => {
			console.log('Event clicked!', event);
			selectEvent(event);
		},
		datesSet: (dateInfo) => {
			console.log('Dates set! (calendar view initialised)');
		}
	};

	let shelvingData = { title: 'Shelving', duration: '00:30', color: 'red' };
	let helpdeskData = { title: 'Helpdesk', duration: '01:00', color: 'green' };

	function extractEvents() {
		console.log('Events');
		console.log(JSON.stringify(calendarComponentRef.getAPI().getEvents()));
		console.log('Resources');
		console.log(JSON.stringify(calendarComponentRef.getAPI().getResources()));
		console.log('Events from Resources');
		calendarComponentRef
			.getAPI()
			.getResources()
			.forEach((resource) => {
				console.log(JSON.stringify(resource.getEvents()));
			});
	}

	function selectEvent(eventWrapper) {
		deselectEvent();
		selectedEventWrapper = eventWrapper;
		selectedEventWrapper.el.style.borderColor = 'black';
	}

	function deselectEvent() {
		// Set the border colour back to where it was.
		if (selectedEventWrapper) {
			selectedEventWrapper.el.style.borderColor =
				selectedEventWrapper.event.borderColor;
			selectedEventWrapper = undefined;
		}
	}

	function deleteSelectedEvent() {
		if (selectedEventWrapper) {
			selectedEventWrapper.event.remove();
		}
	}

	function persistEvents() {
		let eventsToPersist = calendarComponentRef
			.getAPI()
			.getEvents()
			.map((event) => {
				// Strip out all the object cruft
				let parsedEvent = JSON.parse(JSON.stringify(event));

				// Manually add the resource ID to the event
				parsedEvent.resourceId = event.getResources()[0].id;

				return parsedEvent;
			});

		localStorage.setItem('events', JSON.stringify(eventsToPersist));
	}
</script>

<main>
	<h1>Library Time</h1>
	<div class="activityGroup">
		<div class="shelving">
			<Draggable
				eventData={shelvingData}
				class="draggable"
				style="background-color: red;">Shelving</Draggable
			>
		</div>
		<div class="helpdesk">
			<Draggable
				eventData={helpdeskData}
				class="draggable"
				style="background-color: green;">Helpdesk</Draggable
			>
		</div>
	</div>
	<button on:click={deleteSelectedEvent}>Delete Selected Event</button>
	<button on:click={deselectEvent}>Deselect Event</button>
	<button on:click={extractEvents}>Extract Events</button>
	<FullCalendar bind:this={calendarComponentRef} {options} />
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		width: 900px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

	.activityGroup {
		display: flex;
		flex-direction: row;
	}

	:global(.draggable) {
		color: white;
		background: #3788d8;
		width: fit-content;
		padding: 1rem;
		cursor: pointer;
	}
</style>
