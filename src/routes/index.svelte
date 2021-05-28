

<div class="row">

	<div class="col">
		
		<div class="row">
			<div class="col">
				<label class="form-label">Dgraph URL</label>
				<input type="text" name="dgraph-endpoint" class="form-control" bind:value={dgraphEp} on:change={epchange}>
			</div>
			<div class="col">
				<label class="form-label">Dgraph Action</label>
				<select class="form-select" bind:value={dgraphAction}>
					<option selected value="query">Query</option>
					<option value="mutate">Mutate</option>
					<option value="alter">Alter</option>
				</select>
			</div>
			<div class="col">
				<label class="form-label">Content Type</label>
				<select class="form-select" bind:value={dgraphContentType}>
					<option selected value="application/rdf">application/rdf</option>
					<option value="application/json">application/json</option>
					
				</select>
			</div>
		</div>

		<div class="form-floating">
			<textarea class="form-control mt-1" style="height: 400px" bind:value={input}></textarea>
			<label>Input</label>
		</div>

		<button class="btn btn-success float-end mt-1" on:click={run}>Run</button>
	</div>

	<div class="col">

		

		<div class="form-floating">
			<textarea class="form-control mt-1" style="height: 400px" bind:value={output}></textarea>
			<label>Output</label>
		</div>
	</div>

</div>


<script type="ts">

	let dgraphEp: string = 'http://localhost:8080';
	let input: string;
	let dgraphAction: string;
	let output: string;
	// let dgraphContentTypes = [
	// 	'application/rdf',
	// 	'application/json'
	// ]
	let dgraphContentType: string = 'application/rdf';

	async function query(input) {
		const response = await fetch(`${dgraphEp}/query?ro=true&timeout=20s`, {
			method: 'POST', // *GET, POST, PUT, DELETE, etc.
			headers: {
				'Content-Type': 'application/dql'
			},
			body: input
		});
		return response.json();
	}

	async function mutate(input) {
		const response = await fetch(`${dgraphEp}/mutate?commitNow=true`, {
			method: 'POST', // *GET, POST, PUT, DELETE, etc.
			headers: {
				'Content-Type': dgraphContentType, //'application/rdf'
			},
			body: input
		});
		return response.json();
	}

	async function alter(input) {
		const response = await fetch(`${dgraphEp}/alter`, {
			method: 'POST', // *GET, POST, PUT, DELETE, etc.
			// headers: {
			// 	'Content-Type': 'application/rdf'
			// },
			body: input
		});
		return response.json();
	}

	// function initdgraph() {
	// 	const clientStub = new dgraph.DgraphClientStub(
	// 		// addr: optional, default: "http://localhost:8080"
	// 		dgraphEp,
	// 		// legacyApi: optional, default: false. Set to true when connecting to Dgraph v1.0.x
	// 		false,
	// 	);
	// 	return new dgraph.DgraphClient(clientStub);
	// }

	
	// let dgraphClient = initdgraph()

	async function run() {
		console.log(`run clicked: ep:${dgraphEp} - in:${input} - act:${dgraphAction}`)


		switch(dgraphAction) {
			case "query":
				output = JSON.stringify(await query(input), null, 4)
				break;
			case "mutate":
				output = JSON.stringify(await mutate(input), null, 4)
				break;
			case "alter":
				output = JSON.stringify(await alter(input), null, 4)
				// dgraphClient.alter()
				break;
			default:
				alert(`unexpected action '${dgraphAction}'`)
		}
	}
	function epchange() {
		// dgraphClient = initdgraph()
	}
</script>




