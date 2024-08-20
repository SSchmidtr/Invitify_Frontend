<script>
	import { onMount } from 'svelte';
	import { fade } from 'svelte/transition';
	import Button from '@smui/button';

  
	let events = [];
	let currentEvent = null;
	let newEventName = '';
	let invitationText = '';
	let guests = [];
	let sentInvitations = 0;
	let confirmedInvitations = 0;
	let pendingInvitations = 0;
	let declinedInvitations = 0;
	let activeTab = 'dashboard';
  
	const sidebarItems = [
	  { icon: 'home', label: 'Dashboard', value: 'dashboard' },
	  { icon: 'calendar', label: 'Events', value: 'events' },
	  { icon: 'users', label: 'Guests', value: 'guests' },
	  { icon: 'bell', label: 'Notifications', value: 'notifications' },
	  { icon: 'settings', label: 'Settings', value: 'settings' },
	];
  
	function handleFileUpload(event) {
	  const file = event.target.files[0];
	  // Simulate guest loading with status
	  guests = [
		{ name: 'Guest 1', phone: '+1234567890', status: 'pending' },
		{ name: 'Guest 2', phone: '+0987654321', status: 'pending' },
		{ name: 'Guest 3', phone: '+1122334455', status: 'pending' },
		{ name: 'Guest 4', phone: '+5566778899', status: 'pending' },
	  ];
	  pendingInvitations = guests.length;
	}
  
	function sendInvitations() {
	  sentInvitations = guests.length;
	  pendingInvitations = sentInvitations;
	alert(`Sent ${sentInvitations} invitations.`);
	}
  
	function createEvent() {
	  if (newEventName.trim() === '') {
		alert('Please enter an event name.');
		return;
	  }
	  events = [...events, { name: newEventName, id: Date.now() }];
	  newEventName = '';
	}
  
	function viewEvent(eventId) {
	  currentEvent = events.find(e => e.id === eventId);
	  activeTab = 'events';
	  sentInvitations = 0;
	  confirmedInvitations = 0;
	  pendingInvitations = 0;
	  declinedInvitations = 0;
	}
  
	function clearCurrentEvent() {
	  currentEvent = null;
	}
  
	function updateGuestStatus(guest, status) {
	  guest.status = status;
	  if (status === 'confirmed') {
		confirmedInvitations += 1;
		pendingInvitations -= 1;
	  } else if (status === 'declined') {
		declinedInvitations += 1;
		pendingInvitations -= 1;
	  }
	  guests = [...guests];
	}
	
	function resendInvite(guest) {
	  alert(`Resent invitation to ${guest.name}`);
	}
  
	onMount(() => {
	  const interval = setInterval(() => {
		if (currentEvent && confirmedInvitations < sentInvitations) {
		  const unconfirmedGuests = guests.filter(g => g.status === 'pending');
		  if (unconfirmedGuests.length > 0) {
			updateGuestStatus(unconfirmedGuests[0], 'confirmed');
		  }
		}
	  }, 5000);
  
	  return () => clearInterval(interval);
	});
  </script>
  
  <main class="flex h-screen bg-gray-100">
	<!-- Sidebar -->
	<aside class="w-64 bg-white shadow-md">
	  <div class="p-4">
		<img src="/wedding-planner-logo.png" alt="Wedding Planner Logo" class="w-full mb-4">
	  </div>
	  <nav>
		{#each sidebarItems as item}
		  <button
			on:click={() => activeTab = item.value}
			class="flex items-center w-full p-4 text-left hover:bg-purple-100 {activeTab === item.value ? 'bg-purple-200 text-purple-800' : 'text-gray-700'}"
		  >
			<i class="fas fa-{item.icon} mr-2"></i>
			<span>{item.label}</span>
		  </button>
		{/each}
	  </nav>
	</aside>
  
	<!-- Main content -->
	<div class="flex-1 overflow-y-auto">
	  <header class="bg-white shadow-sm">
		<div class="max-w-7xl mx-auto py-4 px-4 sm:px-6 lg:px-8">
		  <h1 class="text-2xl font-semibold text-gray-900">Wedding Planner Platform</h1>
		</div>
	  </header>
  
	  <main class="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
		{#if activeTab === 'dashboard'}
		  <div class="bg-white shadow rounded-lg p-6" transition:fade>
			<h2 class="text-xl font-semibold mb-4">Event Overview</h2>
			<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
			  <div>
				<h3 class="text-lg font-medium mb-2">Invitation Statistics</h3>
				<div class="h-64 bg-gray-200 rounded-lg flex items-center justify-center">
				  <p class="text-gray-500">Chart placeholder</p>
				</div>
			  </div>
			  <div>
				<h3 class="text-lg font-medium mb-2">Upcoming Events</h3>
				<ul class="space-y-2">
				  {#each events.slice(0, 3) as event}
					<li class="bg-purple-100 p-2 rounded">
					  <span class="font-medium">{event.name}</span>
					</li>
				  {/each}
				</ul>
			  </div>
			</div>
		  </div>
		{:else if activeTab === 'events'}
		  <div class="bg-white shadow rounded-lg p-6" transition:fade>
			<h2 class="text-xl font-semibold mb-4">Events</h2>
			{#if currentEvent === null}
			  <div class="mb-4">
				<input 
				  type="text" 
				  bind:value={newEventName} 
				  placeholder="Enter event name" 
				  class="p-2 border rounded mr-2"
				>
				<Button on:click={createEvent} variant="raised">
					<lable>Create new event</lable>
				  </Button>
			  </div>
			  {#if events.length === 0}
				<p>No events found. Create a new event to get started!</p>
			  {:else}
				<ul class="space-y-2">
				  {#each events as event}
					<li>
					  <button 
						on:click={() => viewEvent(event.id)}
						class="w-full text-left bg-purple-100 p-2 rounded hover:bg-purple-200"
					  >
						{event.name}
					  </button>
					</li>
				  {/each}
				</ul>
			  {/if}
			{:else}
			  <div>
				<button on:click={clearCurrentEvent} class="mb-4 text-purple-600 hover:text-purple-800">
				  &larr; Back to Events
				</button>
				<h3 class="text-lg font-medium mb-2">{currentEvent.name}</h3>
				<div class="space-y-4">
				  <div>
					<h4 class="font-medium">Invitation</h4>
					<textarea 
					  bind:value={invitationText}
					  placeholder="Write your invitation message here..."
					  class="w-full p-2 border rounded"
					  rows="5"
					></textarea>
				  </div>
				  <div>
					<h4 class="font-medium">Guest List</h4>
					<input type="file" on:change={handleFileUpload} accept=".csv,.xlsx,.xls" class="mb-2">
					{#if guests.length > 0}
					  <p>{guests.length} guests loaded</p>
					  <button 
						on:click={sendInvitations} 
						class="bg-purple-600 text-white px-4 py-2 rounded hover:bg-purple-700"
						disabled={!invitationText}
					  >
						Send Invitations
					  </button>
					{/if}
				  </div>
				  <div>
					<h4 class="font-medium">Event Metrics</h4>
					<p>Invitations Sent: <strong>{sentInvitations}</strong></p>
					<p>Confirmations Received: <strong>{confirmedInvitations}</strong></p>
					<p>Pending Invitations: <strong>{pendingInvitations}</strong></p>
					<p>Declined Invitations: <strong>{declinedInvitations}</strong></p>
					<p>Confirmation Progress:</p>
					<progress value={confirmedInvitations} max={sentInvitations} class="w-full"></progress>
				  </div>
				</div>
			  </div>
			{/if}
		  </div>
		{:else if activeTab === 'guests'}
		  <div class="bg-white shadow rounded-lg p-6" transition:fade>
			<h2 class="text-xl font-semibold mb-4">Guest List</h2>
			{#if guests.length === 0}
			  <p>No guests added yet. Upload a guest list from the Events tab.</p>
			{:else}
			  <table class="w-full">
				<thead>
				  <tr>
					<th class="text-left">Name</th>
					<th class="text-left">Phone</th>
					<th class="text-left">Status</th>
					<th class="text-left">Action</th>
				  </tr>
				</thead>
				<tbody>
				  {#each guests as guest}
					<tr>
					  <td>{guest.name}</td>
					  <td>{guest.phone}</td>
					  <td>{guest.status}</td>
					  <td>
						{#if guest.status === 'pending'}
						<button 
						  on:click={() => resendInvite(guest)}
						  class="bg-purple-600 text-white px-4 py-2 rounded hover:bg-purple-700"
						>
						  Resend Invite
						</button>
						{/if}
					  </td>
					</tr>
				  {/each}
				</tbody>
			  </table>
			{/if}
		  </div>
		{:else if activeTab === 'notifications'}
		  <div class="bg-white shadow rounded-lg p-6" transition:fade>
			<h2 class="text-xl font-semibold mb-4">Notifications</h2>
			<p>No new notifications.</p>
		  </div>
		{:else if activeTab === 'settings'}
		  <div class="bg-white shadow rounded-lg p-6" transition:fade>
			<h2 class="text-xl font-semibold mb-4">Settings</h2>
			<p>Platform settings will be available here.</p>
		  </div>
		{/if}
	  </main>
	</div>
  </main>
  
  <style>
	@import 'https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.3/css/all.min.css';
	@import 'https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css';
  
	:global(body) {
	  font-family: 'Arial', sans-serif;
	}
  
	progress {
	  height: 20px;
	  border-radius: 4px;
	}
  
	progress::-webkit-progress-bar {
	  background-color: #e9ecef;
	  border-radius: 4px;
	}
  
	progress::-webkit-progress-value {
	  background-color: #8b5cf6;
	  border-radius: 4px;
	}
  </style>