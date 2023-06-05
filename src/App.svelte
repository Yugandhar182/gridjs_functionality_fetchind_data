<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";
	import 'bootstrap/dist/css/bootstrap.min.css';
  
	let jsonData = [];
	let sortedData = [];
	let searchQuery = "";
	let tableVisible = false;
	let selectedCandidate = null;
	let currentPage = 1;
	let sortAsc = true; // Track the sort order
	const itemsPerPage = 10;
  
	const dispatch = createEventDispatcher();
  
	onMount(async () => {
	  await fetchData();
	  tableVisible = true;
	});
  
	async function fetchData() {
	  const response = await fetch(`https://api.recruitly.io/api/candidate/?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&search=${encodeURIComponent(searchQuery)}`);
	  const responseData = await response.json();
	  jsonData = responseData.data;
	  sortData(); // Sort the data initially
	}
  
	function handleTitleClick(candidate) {
	  selectedCandidate = candidate;
	  dispatch("showPopup");
	}
  
	function sortData() {
	  if (sortAsc) {
		sortedData = jsonData.sort((a, b) => a.firstName.localeCompare(b.firstName));
	  } else {
		sortedData = jsonData.sort((a, b) => b.firstName.localeCompare(a.firstName));
	  }
	}
  
	function toggleSortOrder() {
	  sortAsc = !sortAsc;
	  sortData();
	}
  
	$: filteredData = sortedData.filter(candidate => candidate.firstName.toLowerCase().includes(searchQuery.toLowerCase()));
  
  </script>
  
  <style>
	.popup {
	  position: fixed;
	  top: 0;
	  left: 0;
	  width: 100%;
	  height: 100%;
	  background-color: rgba(0, 0, 0, 0.5);
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
  
	.popup-content {
	  background-color: rgb(242, 243, 244);
	  padding: 20px;
	  border-radius: 5px;
	}
  </style>
  
  <main class="container mt-4">
	<input type="text" bind:value={searchQuery} placeholder="Search..." class="form-control mb-3" />
	<button class="btn btn-primary mb-3" on:click={() => (currentPage = 1)}>Search</button>
  
	<div class="mb-3">
	  <button class="btn btn-primary" on:click={toggleSortOrder}>Sort {sortAsc ? 'Asc' : 'Desc'}</button>
	</div>
  
	{#if tableVisible}
	  <table class="table">
		<thead class="thead-light">
		  <tr>
			<th>First Name</th>
			<th>Surname</th>
			<th>Email</th>
			<th>Mobile</th>
		  </tr>
		</thead>
		<tbody>
		  {#each filteredData.slice((currentPage - 1) * itemsPerPage, currentPage * itemsPerPage) as candidate}
			<tr>
			  <td>
				<a href="#" on:click|preventDefault={() => handleTitleClick(candidate)}>{candidate.firstName}</a>
			  </td>
			  <td>{candidate.surname}</td>
			  <td>{candidate.email}</td>
			  <td>{candidate.mobile}</td>
			</tr>
		  {/each}
		</tbody>
	  </table>
  
	  <ul class="pagination">
		{#each Array(Math.ceil(filteredData.length / itemsPerPage)).fill() as _, i}
		  <li class="page-item {currentPage === i + 1 ? 'active' : ''}">
			<a class="page-link" href="#" on:click|preventDefault={() => (currentPage = i + 1)}>{i + 1}</a>
		  </li>
		{/each}
	  </ul>
	{/if}
  </main>
  
  {#if selectedCandidate}
  <div class="popup">
	<div class="popup-content">
	  <h2>{selectedCandidate.firstName}</h2>
	  <p>ID: {selectedCandidate.id}</p>
	  <p>First Name: {selectedCandidate.firstName}</p>
	  <p>Surname: {selectedCandidate.surname}</p>
	  <p>Email: {selectedCandidate.email}</p>
	  <button class="btn btn-primary" on:click={() => (selectedCandidate = null)}>Close</button>
	</div>
  </div>
  {/if}
  