


<div class="container mb-5">

	<h1>Database Schema</h1>

	<div class="row">
		<div class="col">
			
			{#await dataPromise}
				<!-- promise is pending -->
				<p>waiting for the promise to resolve...</p>
			{:then value}
				<!-- promise was fulfilled -->
				<h2>Types</h2>
				<ul>
					{#each value.data.types as type}
						<li>{type.name} - {type.fields.length}
							<button class="btn btn-success" on:click={e => create(type.name)}>Create</button>
							<button class="btn btn-success" on:click={e => list(type.name)}>List</button>
							<ul>
								{#each type.fields as field}
								<li>{buildFieldString(field)}</li>
								{/each}
							</ul>

						</li>
					{:else}
						<li>No Types</li>
					{/each}
				</ul>

				<details>
					<summary>JSON of Types</summary>
					<pre>{JSON.stringify(value.data.types, null, 4)}</pre>
				</details>


				<h2>Predicates</h2>
				<ul>
					{#each value.data.schema as item}
						<li>{buildFieldString(item)}
						</li>
					{:else}
						<li>No Types</li>
					{/each}
				</ul>

				<details>
					<summary>JSON of Predicates</summary>
					<pre>{JSON.stringify(value.data.schema, null, 4)}</pre>
				</details>

				
			{:catch error}
				<!-- promise was rejected -->
				<p>Something went wrong: {error.message}</p>
			{/await}
		</div>
		<div class="col">
			<h2>Results</h2>
			<textarea bind:value={result} class="form-control mt-1" style="height: 400px"></textarea>
			<hr>
			<form>
				{#each fields as field}
					<div class="form-group">
						<label for="{field.name}_id">{field.name}</label>
						{#if field.list}
						<b>Note lists not supported</b>
						{/if}
						{#if field.type == "string"}
						<input type="text" class="form-control" id="{field.name}_id">
						{:else if field.type == "datetime"}
						<input type="datetime-local" class="form-control" id="{field.name}_id">
						{:else }
						<input type="text" class="form-control" id="{field.name}_id">
						{/if}
					</div>
				{:else}
					<h4>Click create next to a type to load the corresponding form.</h4>
				{/each}
			</form>
		</div>
	</div>
</div>

<script type="text/javascript">

	let dgraphEp = 'http://localhost:8080';
	let input;
	let dgraphAction;
	let output;
	export let result = "";
	export let fields = [
		// {
		// 	"name": "name",
		// 	"type": "string"
		// }
	];

	export function buildFieldString(field) {
		let idx = ""
		if (field.index) {
			idx = `@index(${field.tokenizer.join(",")}) `
		}
		if (field.list) {
			field.type = `[${field.type}]`
		}
		return `${field.name || field.predicate} ${field.type} ${idx}.`
	}

	async function query(input) {
		const response = await fetch(`${dgraphEp}/query?ro=true&timeout=20s`, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/dql'
			},
			body: input
		});
		return response.json();
	}

	async function mutate(input) {
		const response = await fetch(`${dgraphEp}/mutate?commitNow=true`, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/rdf'
			},
			body: input
		});
		return response.json();
	}

	async function alter(input) {
		const response = await fetch(`${dgraphEp}/alter`, {
			method: 'POST',
			body: input
		});
		return response.json();
	}



	export const schema = query(`schema {
		type
		index
		reverse
		tokenizer
		list
		count
		upsert
		lang
	}`)

	async function buildTypeMap(schemaPromise) {
		let s = await schemaPromise;

		const map = {}

		// console.log(s.data.schema)
		for (let entry of s.data.schema) {
			entry = Object.assign({}, entry)
			let name = entry.predicate
			delete entry.predicate
			map[name] = entry
		}
		console.log(map)

		return map
	}

	export const typeMap = buildTypeMap(schema);


	async function buildData(schemaPromise, typeMapPromise) {
		let schema = await schemaPromise;
		// console.log("SCHEMA:", schema)
		let typeMap = await typeMapPromise;


		for (var typeIdx = 0; typeIdx < schema.data.types.length; typeIdx++) {
			let type = schema.data.types[typeIdx]

			for (var fieldIdx = 0; fieldIdx < type.fields.length; fieldIdx++) {
				let field = type.fields[fieldIdx]

				let fieldname = field.name
				if (field.name.startsWith('~')) {
					fieldname = field.name.substring(1) //remove first character for lookup
				}

				let merged = Object.assign({}, field, typeMap[fieldname])
				// console.log(merged)//, schema.data.types[typeIdx])
				schema.data.types[typeIdx].fields[fieldIdx] = merged
			}
		}

		// let typeIdx = 0
		// for (let type of schema.data.types) {
		// 	let idx = 0
		// 	for (let field of type.fields) {
		// 		if (!typeMap[field.name]) {idx++; continue} //remove ~
				
		// 		let merged = Object.assign({}, field, typeMap[field.name])
		// 		console.log(merged)//, schema.data.types[typeIdx])
		// 		field = merged

		// 		// schema.data.types[type.name].fields[idx] = 
		// 		idx ++
		// 	}
		// 	typeIdx ++
		// }

		// console.log("AFTER:", schema)
		return schema
	}

	export const dataPromise = buildData(schema, typeMap);

	async function create(typeName) {
		let schema = await dataPromise
		console.log("schema:", schema)

		let typedef = schema.data.types.find(type => type.name == typeName);

		fields = typedef.fields

		console.log("create:", typeName, typedef)
	}
	async function list(typeName) {
		console.log("list:", typeName)
		result = await query(`{
			q(func: type(${typeName})) {
				expand(_all_)
			}
		}`)
		result = JSON.stringify(result.data, null, 2)
	}

	
</script>
