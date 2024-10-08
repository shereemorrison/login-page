<script lang="ts">
	import { createClient } from '@supabase/supabase-js';
	import supabase from '$lib/supabase';

	let products: Array<{ name: string; stock: number }> = [];
	let errorMessage: string = '';
	let isLoggedIn: boolean = false;
	let email: string = '';
	let password: string = '';

	// Check Auth Status on Page Load
	async function checkAuthStatus() {
		const { data: { session } } = await supabase.auth.getSession();
		isLoggedIn = !!session;
		if (isLoggedIn) {
			console.log("User is logged in");
		} else {
			console.log("User is not logged in");
		}
	}

	checkAuthStatus();

	// Login Function
	async function handleLogin() {
		try {
			const { data, error } = await supabase.auth.signInWithPassword({ email, password });

			if (error) {
				errorMessage = `Login failed: ${error.message}`;
				console.error('Login error:', error);
				return;
			}

			isLoggedIn = true;
			errorMessage = 'Login successful';
			console.log('Login successful');
		} catch (error) {
			errorMessage = `Unexpected error: ${(error as Error).message || 'Unknown error'}`;
			console.error('Unexpected error:', error);
		}
	}

	// Sign in with GitHub
	async function signInWithGithub() {
		const { data, error } = await supabase.auth.signInWithOAuth({
			provider: 'github',
		});

		if (error) {
			errorMessage = `GitHub login failed: ${error.message}`;
			console.error('GitHub login error:', error);
			return;
		}

		isLoggedIn = true;
		errorMessage = 'GitHub login successful';
		console.log('GitHub login successful');
	}

		// Logout Function
	async function handleLogout() {
		try {
			await supabase.auth.signOut();
			isLoggedIn = false;
			products = [];
			errorMessage = 'Logged out successfully.';

			// Clear any local storage data to ensure the session is forgotten
			localStorage.removeItem('supabase.auth.token');

			console.log('Logout successful, GitHub session cleared');
		} catch (error) {
			errorMessage = `Error logging out: ${(error as Error).message || 'Unknown error'}`;
			console.error('Logout error:', error);
		}
	}


	// Fetching Products Function
	async function getProducts() {
		if (!isLoggedIn) {
			errorMessage = 'You must be logged in to view the inventory';
			return;
		}

		try {
			const { data, error } = await supabase
				.from('Products')
				.select('*');

			if (error) {
				errorMessage = `Error fetching products: ${error.message}`;
				console.error('Error fetching products:', error);
				return;
			}

			products = data;
			console.log('Products data:', data);
		} catch (error) {
			errorMessage = `Error fetching products: ${(error as { message: string }).message || 'Unknown error'}`;
			console.error('Unexpected error:', error);
		}
	}
</script>


  <!-- Tailwind -->
<div class="container h-full mx-auto flex justify-center items-center">
	<div class="login-box">
	  <form on:submit|preventDefault={handleLogin}>
		<input type="email" bind:value={email} placeholder="Email" required>
		<input type="password" bind:value={password} placeholder="Password" required>
		{#if !isLoggedIn}
		  <button type="submit" class="btn variant-filled">Login with Email</button>
		  <button type="button" class="btn variant-filled" on:click={signInWithGithub}>Login with GitHub</button>
		  <button type="button" class="btn variant-filled" on:click={getProducts}>View Inventory</button>
		{/if}
		{#if isLoggedIn}
		  <button type="button" class="btn variant-filled" on:click={handleLogout}>Logout</button>
		  <button type="button" class="btn variant-filled" on:click={getProducts}>View Inventory</button>
		{/if}
		{#if errorMessage}
		  <p id="error-message" class="error-message">{errorMessage}</p>
		{/if}
	  </form>
	  {#if isLoggedIn && products.length > 0} <br>
		

<div class="table-container">
	<table class="table table-hover">
		<thead>
			<tr>
				<th>Product</th>
				<th>stock</th>
				
			</tr>
		</thead>
		<tbody>
			{#each products as product}
				<tr>
					<td>{product.name}</td>
					<td>{product.stock}</td>
				
				</tr>
			{/each}
		</tbody>
	</table>
</div>
{/if}
</div>
</div>
  
<style>
	.login-box {
	  margin-top: 100px;
	  color: black;
	}
  
	button {
	  color: white;
	  margin-left: 10px;
	}

  
	.error-message {
	  color: red;
	}

	th, td {
		color:white;
		text-align: center;
	}
</style>
