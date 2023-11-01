<script>

   //import { plant_1, plant_2 } from '../variables.js'
   //let name_image = 'nombre de archivo'
   let view_image
   let modal = false
   let load = false
   let resultAPI = []//plant_1.result.classification.suggestions


   function updateImagen(event){
      event.preventDefault()
      let file_image = event.target.files[0]
      view_image = URL.createObjectURL(file_image)
      //name_image = file_image.name
      //console.info(name_image)
      
   }

   function identifyPlant(e){
      e.preventDefault()
      console.info('Enviando, por favor espere un momento...')
      let formData = new FormData(e.target)
      formData.append('similar_images', 'true')

      let options = {
         method: 'POST',
         headers: {
            'Api-Key': import.meta.env.VITE_API_KEY
         },
         body: formData
      }

      load = true
      return fetch(import.meta.env.VITE_API_URL, options).then(response=>response.json()).then(json=> {
         resultAPI = json.result.classification.suggestions
         modal = !modal
         load = false
      } ).catch(error=>console.errr(error))

      //return fetch('https://jsonplaceholder.typicode.com/todos').then(response=>response.json()).then(json=> console.info(json) ).catch(error=>console.error(error))
   }

</script>

<div class='relative top-0 left-0 flex flex-col justify-center items-center gap-y-3 h-full'>
   <h2 class='font-semibold tracking-tighter text-xl text-[#232E21] text-center uppercase'>imagen a identificar</h2>
   <figure class='bg-zinc-100 w-56 h-56 overflow-hidden border-2 border-zinc-100 shadow-sm'>
      {#if view_image}
         <img id='img' alt='plant' class='object-cover w-[inherit] h-[inherit] z-0' src={view_image}/>
      {:else}
         <img id='img' alt='plant' class='object-cover w-[inherit] h-[inherit] z-0' src='/template.jpg'/>
      {/if}
   </figure>
   <div>
      <form id='formulario' on:submit|preventDefault={ identifyPlant } class='flex items-center justify-center'>
         <label for='imagen' class='font-semibold tracking-tight text-xs text-zinc-500 text-center uppercase p-2 border-2 cursor-pointer box-content'>Cargar imagen</label>
         <input type='file' on:change={ updateImagen } class='hidden' id='imagen' name='imagen'/>
         <button disabled={!view_image} class='font-semibold tracking-tight text-xs px-5 py-2 text-center uppercase bg-lime-500 text-lime-900' type='submit'>identificar</button>
      </form>
   </div>
   <div class:hidden={!load}>
      <span class='text-sm'><i class='bx bx-loader-alt animate-spin mr-2 text-lg'></i>Espera un momento por favor...</span>
   </div>
   {#if modal}
      <div class='fixed top-0 h-full w-full overflow-y-scroll flex flex-col bg-zinc-100 p-5'>
         <span class='absolute top-3 right-4'><i on:click={ ()=> modal = false } on:keydown={ ()=> modal = false } class='bx bxs-x-circle text-4xl text-lime-500 cursor-pointer'></i></span>
         <h2 class='text-2xl uppercase text-zinc-500 font-bold tracking-tighter border-b-2 pb-4'>Información</h2>
         <div class='flex flex-col sm:flex-row w-[inherit] gap-x-3 py-4'>
            <figure class='w-56 h-56 sm:w-40 sm:h-40 shrink-0'>
               <img class='object-cover w-[inherit] h-[inherit]' src={view_image } alt='mi plant'>
            </figure>
            <div class='w-[inherit]'>
               <div class='border-b-2 pt-2'>
                  <span class='text-sm text-zinc-400 uppercase tracking-tight font-semibold'>nombre</span>
                  <p class='pb-2 text-lime-500 font-semibold text-2xl'>{ resultAPI[0].name }</p>
               </div>
               <div class='border-b-2 pt-2'>
                  <span class='text-sm text-zinc-400 uppercase tracking-tight font-semibold'>probabilidad</span>
                  <p class='pb-2 font-medium text-xl'>{ (resultAPI[0].probability).toFixed(2) }%</p>
               </div>
            </div>
         </div>
         <div class='border-b-2 py-2'>
            <span class='text-sm text-zinc-400 uppercase tracking-tight font-semibold'>imagenes similares</span>
            <div class='py-2 font-medium text-xl flex flex-wrap gap-3'>
               {#each resultAPI[0].similar_images as image}
               <img class='w-40' src={ image.url } alt='similar'>
               {/each}
            </div>
         </div>
         <div class='border-b-2 pt-2'>
            <div>
               <span class='text-sm text-zinc-400 uppercase tracking-tight font-semibold'>descripción</span>
               <p class='py-2 tracking-wide text-2xl'>{ resultAPI[0].details.description.en ? resultAPI[0].details.description.en.value:'No hay descripción.'}</p>
            </div>
            <div class='py-2'>
               <p class='capitalize font-medium'>más informacion</p>
               <ul>
               {#each Object.values(resultAPI[0].details.url) as url}
                  <li><a class='text-sky-600 {url === null ? "hidden":"block"}' href={url} target='_blank'>{url}</a></li>
               {/each}
               </ul>
            </div>
         </div>
         <div class='border-b-2 py-2'>
            <span class='text-sm text-zinc-400 uppercase tracking-tight font-semibold'>taxonomía</span>
            <ul class='flex flex-wrap gap-3 py-2'>
            {#each Object.entries(resultAPI[0].details.taxonomy) as e,i}
               <li class='flex w-40 h-20 flex-col items-center justify-center text-xl bg-lime-100'><span class='text-sm text-lime-500 uppercase tracking-tight font-semibold mx-2'>{e[0]}</span> {e[1]}</li>
            {/each}
            </ul>
         </div>
      </div>
   {/if}
</div>