<script lang="ts">
	import { onMount } from 'svelte'
	let width = 320,
		height = 240
	let vid: HTMLVideoElement
	let canvasIn: HTMLCanvasElement
	let canvasOut: HTMLCanvasElement

	let process = () => {}
    let arr = []
    let transform = []
    const r = 8
    const x = width / r

	onMount(() => {
        console.log(vid.offsetHeight, vid.offsetWidth)
        canvasIn.width = width
        canvasIn.height = height
        // canvasOut.width = width
        // canvasOut.height = height
		process = () => {
			const ctxIn = canvasIn.getContext('2d')
			// const ctxOut = canvasOut.getContext('2d')

			ctxIn.drawImage(vid, 0, 0, width, height)
			const frame = ctxIn.getImageData(0, 0, width, height)
			const data = frame.data
			transform = []

			// for (let i = 0; i < data.length; i += 4) {
			// 	data[i+2] = 0
			// }

            // for(let i = 0; i < data.length; i += 8) {
            //     for(let j = 0; j < height; j += 2) {
            //         transform.push((data[i] + data[i + 4] + data[i + width] + data[i + width + 4]) / 4)
            //         transform.push((data[i] + data[i + 5] + data[i + width] + data[i + width + 5]) / 4)
            //         transform.push((data[i] + data[i + 6] + data[i + width] + data[i + width + 6]) / 4)
            //         transform.push((data[i] + data[i + 7] + data[i + width] + data[i + width + 7]) / 4)
            //     }
            // }

            const w = (width*4)
            for (let i = 0; i < data.length - (w * r); i += 4 * r) {
                for(let j = 0; j < 4; j++) {
                    let sum = 0
                    for(let k=0;k<r;k++) {
                        for(let l=0;l<r;l++)
                            sum += data[i+j+(k*w)+(l*4)]
                    }
                    sum = (sum) / (r * r)
                    if (Math.ceil(i / w) % r === 0) if (j !== 1) transform.push(sum)
                      else transform.push(data[i+j]) //

                    // data[i+j] = (data[i+j] + data[i+j+4] + data[i+j+w] + data[i+j+w+4]) / 4

                    for(let k=0;k<r;k++) {
                      for(let l=0;l<r;l++)
                        data[i+j+(k*w)+(l*4)] = sum
                    }
                }
            }

            // arr = data
			// ctxOut.putImageData(frame, 0, 0)
			const u8transform = new Uint8ClampedArray(transform)
			// console.log(u8transform.length)
			// ctxOut.putImageData(new ImageData(u8transform, 40), 0, 0)
		}
		navigator.mediaDevices
			.getUserMedia({ video: true, audio: false })
			.then((stream) => {
				vid.srcObject = stream
				vid.play()
			})
			.catch((err) => {
				console.error(`An error occurred: ${err}`)
			})
		setInterval(() => {
			process()
		}, 100)
	})

	const getL = (r, g, b) => {
	 return (0.2126*r + 0.7152*g + 0.0722*b)
	}
	const getChar = (r, g, b) => {
	   const l = getL(r, g, b)
	   // return Math.round(l)
	   // return [".", "-", "+", "&", "#"].reverse()[Math.floor((l * 5) / 255)]
	   // return ["A", "B", "W", "E", "X"].reverse()[Math.floor((l * 5) / 255)]
	   const chars = "@K☻ฌGQ︎#3⌘MกA⌫&%g!เ®︎+*~.".split("") // "ABCDEFGHIJKLMONOPQRSTUVWXYZ".split("")
	   // const emoji = ["@", "K", "🥺", "ฌ", "G", "Q", "🪐", "#", "3", "M", "ก", "🤘", "A", "&", "%", "g", "!", "เ", "+", "🧋", "*", "~", "."]
	   return chars[Math.floor((l * chars.length) / 255)]
	}
</script>

<div class="flex-col gap-4">
	<video
		playsinline
		id="video"
		bind:this={vid}
        width={parseInt(width / 1.5)}
        height={parseInt(height / 1.5)}
        class="fixed top-0 left-0 p-4"
	>
		Video stream not available.
		<track kind="captions" />
	</video>
	<canvas
        bind:this={canvasIn}
        class="border object-cover absolute top-0 left-0 -z-10 opacity-0"
        style={`width:${width}px;height:${height}px`}
    >
		in
	</canvas>
	<!-- <div class="w-[100vw] h-[100svh] bg-neutral-900/10">
	   {arr?.[0]}
	</div> -->
	<!-- <canvas
		bind:this={canvasOut}
		class="border object-cover"
		style={`width:${width}px;height:${height}px`}
	>
		out
	</canvas> -->
	<div class="">
	{#each [...Array(parseInt(height/r)-1).keys()].map(x => x * 4) as i}
	   <div class="flex justify-center items-center">
	   {#each [...Array((parseInt(width/r))).keys()].map(x => x * 4) as j}
				<div
				    class="aspect-square flex justify-center w-10 items-center overflow-hidden"
					style={`
					  background-color: rgba(${transform[(i*x)+j]},${transform[(i*x)+j+1]},${transform[(i*x)+j+2]},0.5);
					  border: 1px solid rgba(${transform[(i*x)+j]},${transform[(i*x)+j+1]},${transform[(i*x)+j+2]},0.8);
					  color: rgb(${transform[(i*x)+j]} ${transform[(i*x)+j+1]} ${transform[(i*x)+j+2]});
					  font-size: ${getL(transform[(i*x)+j], transform[(i*x)+j+1], transform[(i*x)+j+2]) > 100 ? "1" : "2"}em;
					`}
				>{getChar(transform[(i*x)+j], transform[(i*x)+j+1], transform[(i*x)+j+2]) || "☻"}</div>
		{/each}
		</div>
	{/each}
	</div>
</div>
