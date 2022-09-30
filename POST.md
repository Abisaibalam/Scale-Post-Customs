# POST PROCESS - REGREX/SELECTORES

### *BREADCRUMBS*
- Eliminar espacios en TextContent
```javascript
match: \n+
replace: nada 
```
### *MONEY*
- Curency esta en el html y no funciona selector 
	`` meta[property="og:price:currency"]
	``
- price esta en el html y no funciona selector 
	`` [property="og:price:amount"]
	// content
	``
	 
### *IMAGES*
- Selectores para sacar la main

	 `` Selector: featured_image
	 ``
 
 	`` Selector: variantData.featured_image.src
	 ``

- Ejemplo de excluir una imagen con src de su Nombre

 	`` .home-left img:not([src*="MonkeyBackGuarantee"])
	``
	
- Selector cuando extraes imagenes de las thumb y hay paginas que no tienen thumb

	 ``html:not(:has(.product-single__thumbnails))	
	 `` 			
	 
	 poner antes del selector
	 
`Regrex`

- Regrex para arreglar imágenes en result
```
Match:  lighter/
Replace: lighter/.shg
```
### *Aditional Section*
`Paragraph`
- Toma todo antes lo que es acordeones y lo borra 
```
(?=<button class=\"collapsible\">)([\s\S])+
```
- Toma lo que es acordeon y en teoria deberia no mostrarlo 
```
([\s\S]+)(?=PATRON)(ejemplo)
([\s\S]+)(?=class="collapsible")
```
`Tablas`
- Eliminar tablas del pdp
```
Match: (?<=<table)([\s\S])+(?<=</table)
Replace: nada 
```
- Para quitar size chart
```
([\s\S]+)(?=<strong>MEASUREMENTS</strong>)
```
- Para quitar size chart con diferentes titulos 
```
([\s\S]+)(?=<tbody>)
```
`Bullets`
- Arreglar bullets de listas
```
Match: <ul>\n<li>
Replace: <ul> <br> <li>

Match: <li
Replace: <br
```
- Para quitar excesos de espacio en las bullets y dejarlo alinado (regex match and replace)
```
[\s]+[/]+[\s]*
Replace: espacio/espacio 
```
# CUSTOMS
### BRAND
`BRAND`
- Esperar a la brand si no buscarla en el texto
```javascript
async(brand,{input,config},{_,Errors}) =>{
    return await input.page.evaluate(async() => {
        if(brand.includes('Brand:') == false ){
            let url = window.location.href.replace(/\?variant=.*/g,'.json')
            return await fetch(url).then(res => res.json()).then(data => data.product.vendor)
        }else{
            brand = brand.substring(brand.indexOf('Brand:')+7,brand.length)
            brand = brand.substring(0,brand.indexOf('\n'))
            return brand
        }
        
    })
}
```
- Sacar la brand de la PDP y si no existe, del json
```javascript
async (data, {input, config}, {_, Errors}) => {
  let thePDPBrand = await input.page.evaluate(()=>{
  let brandDescription = document.querySelector(".product__section-content .rte")?.textContent
  return brandDescription.match(/(?<=[B|b]rand:)(.+)/gim)?.toString()
  })
  return thePDPBrand?thePDPBrand:data
}
```
### TITLE
- Cambiar Title por amount si el selector esta tomando ejemplo: `Title | $200
```javascript
options => {
    var keys = Object.keys(options)
    var newOptions = {}
    if (keys[0] == null){
        return options
    } else {
        keys.forEach(key => {
            if (options[key].includes('Title') || options[key].includes('Default')){
                options[key] = options[key]
            } else if (options[key].includes("$")) {
                newOptions["Amount"] = options[key]
            }else {
                newOptions[key] = options[key]
            }
        })
        return newOptions
    }
}
```

### MONEY
`CURENCY`
- Currency forzado
```javascript
(data, {input, config}, {_, Errors}) => {
  if(data !== undefined){
  return data}
  else{return "USD"}
}
```
`PRICE`
- Para quitar comas y símbolos de pesos del precio
```javascript
price => Number(price.replace(/\$|,/g, '').trim())
```
`PRICE OR H PRICE`<br>
- Cuando una `,` corta el price y le pone punto
````Javascript
real_price =>{
  try {
      let num = real_price.match(/[0-9.,]+/g)[0];
      if (num.includes(",")){
          num = num.replace(/,/g, "")
      }
      num = Number(num)
      if (num == 0 || isNaN(num)){
          return undefined
      } else{
          return num
       }
  } catch (error){
        return undefined
    }
}
````
- Cuando hay una coma y la tool la pone como un punto
```javascript
real_price =>{
  try {
      let num = real_price.match(/[0-9.,]+/g)[0];
      if (num.includes(",")){
          num = num.replace(/,/g, "")
      }
      num = Number(num)
      if (num == 0 || isNaN(num)){
          return undefined
      } else{
          return num
       }
  } catch (error){
        return undefined
    }
}
```
- Para cuando el higher sale como `0`
```javascript
higher_price =>{
    if(higher_price==0 || higher_price == null){
        return undefined;
}
    else{
        return higher_price
}
return higher_price
}
```
- Cambiar de letra a numero el precio
```javascript
real_price =>{
    
  try {
      let num = real_price.match(/[0-9\.,]+/g)[0];
      if (num.includes(",")){
          num = num.replace(/,/g, "")
      }
      num = Number(num)
      if (num == 0 || isNaN(num)){
          return undefined
      } else{
          return num
       }
    
  } catch (error){
        return undefined
    }
}
```
### OPTIONS
- Mientras la key tenga `size` como texto, te la devolvera como size
```javascript
options => {
    var keys = Object.keys(options)
    const new_options = {...options}
    if (keys[0] == null){
        return options
    } else {
        keys.map(key => {
            if(key.toLowerCase().includes("size")){
                new_options["size"] = options[key]
                delete new_options[key]
            }
        })
        return new_options
   
    }
    return options     
}
```
- Cuando viene `size - color` juntos como opcion `cambia el sentido y el divisor`
```javascript
options => {
    var keys = Object.keys(options)
    var newOptions = {}
    if(keys[0] === null){
        return options
    } else {
        keys.map(key => {
            if(key === "SIZE  COLOR"){
                var val_1 = options[key].split(' - ')
                newOptions["Size"] = val_1[0]
                newOptions["Color"] = val_1[1]
            } else {
                newOptions[key] = options[key]
            }
        })
        return newOptions
    }
}
```
- Cambiar Title por amount si el selector esta tomando ejemplo: `Title | $200
```javascript
options => {
    var keys = Object.keys(options)
    var newOptions = {}
    if (keys[0] == null){
        return options
    } else {
        keys.forEach(key => {
            if (options[key].includes('Title') || options[key].includes('Default')){
                options[key] = options[key]
            } else if (options[key].includes("$")) {
                newOptions["Amount"] = options[key]
            }else {
                newOptions[key] = options[key]
            }
        })
        return newOptions
    }
}
```
- Remover `:` en opcion de color
``` Javascript
(options, {input, config}, {_, Errors}) => {
  var keys = Object.keys(options)
  var newOptions = {}
  if (keys[0] == null){
    return options
  } else {
    keys.forEach(key => {
      if (!options[key].includes('Title') && !options[key].includes('Default')){
        if (key.includes(':')){
          let ky = key.replace(/\n|[\s]{2,}/gmi, '').toUpperCase()
          ky = ky.match(/([\s\S]+)(?=:)/gmi)[0]
          newOptions[ky] = options[key]
        } else {
          newOptions[key] = options[key]
        }
      } 
    })
    return newOptions
  }
}
```
- Para extraer options de route / precios indistintos que no salen en options

``` Javascript
options => {
    var keys = Object.keys(options)
    var newOptions = {}
    if (keys[0] == null){
        return options
    } else {
        keys.forEach(key => {
            if (options[key].includes('Title') || options[key].includes('Default')){
                options[key] = options[key]
            } else if (options[key].includes("$")){
                newOptions = options
            }
              else {
                newOptions[key] = options[key]
            }
        })
        return newOptions
    }
}
``` 

- Para borrar las options de tipo Title: "Default title" pero deja las que sean Title: no sé, precio de gif card

```javascript
options => {
    var keys = Object.keys(options)
    var newOptions = {}
    if (keys[0] == null){
        return options
    } else {
        keys.forEach(key => {
            if (options[key].includes('Title') || options[key].includes('Default')){
                options[key] = options[key]
            } else if (options[key].includes("$")){
                newOptions = options
            }
              else {
                newOptions[key] = options[key]
            }
        })
        return newOptions
    }
}
```
### IMAGES
- Eliminar imagenes que salen como `Background - image : url():`
```javascript
images => {
    if (images[0] == null){
        return []
    } else {
        var i = 0
        images.forEach(x => {
            images[i] = images[i].match(/(//cdn)([\S]+)(.(jpg|png|gif|jpeg|JPG|heic))/g)[0]
            images[i] = "https:" + images[i]
            var resolution = images[i].match(/([0-9]+)x(?=.(jpg|png|gif|jpeg|JPG|heic))/g)
            if (resolution !==null){
              images[i] = images[i].replace(/([0-9]+)x(?=.(jpg|png|gif|jpeg|JPG|heic))/g, "")
            }
            i++
        })
        return images
    }
} 
```
- Sacar imagenes desde el json `si no te sirve elimina la configuracion``Se pega desde la configuracion`
```javascript
"perVariantFns": [
            {
                fnName: 'customFunction',
                fnParams: {
                    fnCode: async (fnParams, page, extractorsDataObj) => {
                        let data = extractorsDataObj.customData
                        extractorsDataObj.customData.customImages = Object.fromEntries(data.variants.map((variant) => {
                            return [variant.id, data.media.filter(media => media['media_type'] == "image" && media.alt.trim() === variant.featured_image.alt.trim()).map((media) => media.src)]
                        }))[data.variantData.id]
                    },
                },
            },
        ]
```
- Agregar un 0 al final de una link image para que no lo borre

```javascript
(data, {input, config}, {_, Errors}) => {
  if (data){
    for(let i = 0; i < data.length; i++){
      let j = data[i].length - 1
      if (data[i][j] == '/'){
        data[i] = data[i] + '0'
      }
    }
  }
  return data
} 

```



### ADITIONAL SECTIONS
`Texto/ Descripciones`
- La descripción está vacía `más funcional`
````javascript
(data, {input, config}, {_, Errors}) => {
  let content = data[0].content
  content = content.replace(/(\n|\s|<\/?[\s\S]+?>)/g, '')
  return content == '' ? []: data
}
````

- Si la descripcion esta vacio
```javascript
(data, {input, config}, {_, Errors}) => {
  let countString = data[0].content.replace(/\s+/g, '').length;
  if (countString >= 87){
      data[0].content = "<b></b>"
      data[0].description_placement = DESCRIPTION_PLACEMENT.DISTANT
      return data
  }else{
  return data
  }
}
```
`Texto/ Acordeones`
- Quitar una sección en específico, Se puede usar para quitar acordeones o tablas de la main description

```javascript
adjacents=>{
    let i=0;
    adjacents.forEach(adjacent=>{
        if(adjacent.content.includes("secciónAFiltrar")){
            adjacents[i].content = adjacents[i].content.replace(/(?=<inicio)([\s\S])+(?<=</fin>)/g, "")
        }
        i++;
    })
    return adjacents
}
```
- Quitar descripciones que están en blanco  (PREPROCESS) `otra si no funcionan`

```javascript
sync ({page}, config)=> {
  await page.evaluate(() => {
    let cont = document.querySelector("Aquí va tu selector")
    if (cont.innerText){
        cont.setAttribute("content", "true")
    }
  })
}

tuselector[content="true"]
```
- Para arreglar puntos de las listas
```javascript
data[i].content = data[i].content.replace(/<li>/g,'<a>')
```
`Broken bullets`
- Arreglar formatos de lista
```javascript
adjacents=>{
    let i=0;
    adjacents.forEach(adjacent=>{
        if(adjacent.content.includes("<ul>")){
            adjacents[i].content = adjacents[i].content.replace(/<ul>/g, "<ul><br>")
        }
        i++;
    })
    return adjacents
}
```
- Arreglar formato de lista `funciona mas`
```javascript
adjacents=>{
    let i=0;
    adjacents.forEach(adjacent=>{
        if(adjacent.content.includes("<li")){
            adjacents[i].content = adjacents[i].content.replace(/<li/g, "<br> <li")
        }
        if(adjacent.content.includes("<ul")){
            adjacents[i].content = adjacents[i].content.replace(/<ul/g, "\n <ul")
        }
        i++;
    })
    return adjacents
}
```
- Para fixear broken lines `para li`

```javascript
adjacents=>{
    let i=0;
    adjacents.forEach(adjacent=>{
        if(adjacent.content.includes("li>")){
            adjacents[i].content = adjacents[i].content.replace(/li>/g, "div>")
        }
        if(adjacent.content.includes("<li")){
            adjacents[i].content = adjacents[i].content.replace(/<li/g, "<div")
        }
        i++;
    })
    return adjacents
}

```
- para arreglar broken bullets (falta de saltos de linea)

```javascript
(data, {input, config}, {_, Errors}) => {
  for(let i in data){
  data[i].content = data[i].content.replace(/<ul>/g,'<ul><br>')
  data[i].content = data[i].content.replace(/<\/li>/g,'</li><br>')
  }
  return data
}
```
`Tablas`
- Dar formato de tabla a span con saltos de linea
```javascript
des => {
    if (des[0] == null){
        return []
    } else {
        var i = 0
        des.forEach(x => {
            if (des[i].content.includes("<table")){
                des[i].content = des[i].content.replace(/<tr/g, '<span')
                des[i].content = des[i].content.replace(/<td/g, '<span')
                des[i].content = des[i].content.replace(/<\/tr>/g, '<br> </span>')
                des[i].content = des[i].content.replace(/<\/td>/g, '</span>')
                des[i].content = des[i].content.replace(/<\/table>/g, '</span>')
                des[i].content = des[i].content.replace(/<\/tbody>/g, '</span>')
                des[i].content = des[i].content.replace(/<table/g, '<span')
                des[i].content = des[i].content.replace(/<tbody/g, '<span')
            }
            if (des[i].content.includes("<ul")){
                des[i].content = des[i].content.replace(/<ul>/g, "<ul><br>")
            }
            i++
        })
        return des
    }
}
```

- Dar formato a una tabla que no tiene su formato tabla por defecto
```javascript
des => {
    if (des[0] == null){
        return []
    } else {
        var i = 0
        des.forEach(x => {
            if (des[i].content.includes("<table")){
                des[i].content = des[i].content.replace(/<tr/g, '<span')
                des[i].content = des[i].content.replace(/<td/g, '<br> <span')
                des[i].content = des[i].content.replace(/<\/tr>/g, '</span>')
                des[i].content = des[i].content.replace(/<\/td>/g, '</span>')
                des[i].content = des[i].content.replace(/<\/table>/g, '</span>')
                des[i].content = des[i].content.replace(/<\/tbody>/g, '</span>')
                des[i].content = des[i].content.replace(/<table/g, '<span')
                des[i].content = des[i].content.replace(/<tbody/g, '<span')
            }
            i++
        })
        return des
    }
}
```
- Dar formato de tabla a una tabla en las descriptions: `opcion 1`
```javascript
 adjacents=>{
     let i=0;
     adjacents.forEach(adjacent=>{
         if(adjacent.content.includes("<tbody")){
             adjacents[i].content = adjacents[i].content.replace(/<p/g, "<span")
             adjacents[i].content = adjacents[i].content.replace(/<\/p>/g, "<\/span>")
             adjacents[i].content = adjacents[i].content.replace(/<table/g, "<div")
             adjacents[i].content = adjacents[i].content.replace(/<\/table>/g, "<\/div>")
             adjacents[i].content = adjacents[i].content.replace(/<tr/g, "<p")
             adjacents[i].content = adjacents[i].content.replace(/<th/g, "<span")
             adjacents[i].content = adjacents[i].content.replace(/<td/g, "<span")
             adjacents[i].content = adjacents[i].content.replace(/<\/tr>/g, "<\/p>")
             adjacents[i].content = adjacents[i].content.replace(/<\/th>/g, "<\/span>")
             adjacents[i].content = adjacents[i].content.replace(/<\/td>/g, "<\/span>")
         }
         if(adjacent.content.includes("<li")){
             adjacents[i].content = adjacents[i].content.replace(/<li/g, "<br> <li")
         }
         i++;
     })
     return adjacents
 }
```
- Reestructurar tablas `opcion 2`

```javascript
adjacents=>{
    let i=0;
    adjacents.forEach(adjacent=>{
        if(adjacent.content.includes("<table")){
            adjacents[i].content = adjacents[i].content.replace(/<table/g, "<div")
            adjacents[i].content = adjacents[i].content.replace(/<\/table>/g, "<\/div>")
            adjacents[i].content = adjacents[i].content.replace(/<tr/g, "<p")
            adjacents[i].content = adjacents[i].content.replace(/<th/g, "<span")
            adjacents[i].content = adjacents[i].content.replace(/<td/g, "<span")
            adjacents[i].content = adjacents[i].content.replace(/<\/tr>/g, "<\/p>")
            adjacents[i].content = adjacents[i].content.replace(/<\/th>/g, "<\/span>")
            adjacents[i].content = adjacents[i].content.replace(/<\/td>/g, "<\/span>")
            adjacents[i].content = adjacents[i].content.replace(/<tbody>/g, "<span>")
            adjacents[i].content = adjacents[i].content.replace(/<\/tbody>/g, "<\/span>")
        }
        i++;
    })
    return adjacents
}
```

- Custom options quitar las tablas de SC `opcion 1`
```javascript
(data, {input, config}, {_, Errors}) => {
  if(data[0]!== null){
    while(data[0].content.includes('<table')){
      data[0].content = data[0].content.replace(/(<table)([\s\S]+)(<\/table>)/g,'')
    }
    return data

  }else{return[]}
}

```
- Post process custom para quitar tablas de las descripciones `opcion 2`

```javascript
adjacents=>{
    let i=0;
    adjacents.forEach(adjacent=>{
        if(adjacent.content.includes("<table")){
            adjacents[i].content = adjacents[i].content.replace(/(<table)([\s\S]+)(<\/table>)/g, "")
        }
        i++;
    })
    return adjacents
}

```

- Quitar tablas de las descripciones y que salga vacía si no hay texto para tomar

```javascript
async (data, {input, config}, {_, Errors}) => {
let descriptionFromPDP = await input.page.evaluate(()=>{
    let descriptionFromPDP = document.querySelector('.sf__accordion-content .ui-accordion-content')?.innerHTML?.replace(/(?=<table)([\s\S])+?(?<=<\/table>)/g, "").replace(/<[^>]+?>/g, "").replace(/(PRODUCT DETAILS|DESCRIPTION|&nbsp;)/gi, "");
    let descriptionFromPDP2 = document.querySelector('.sf__accordion-content')?.innerHTML?.replace(/(?=<table)([\s\S])+?(?<=<\/table>)/g, "").replace(/<[^>]+?>/g, "").replace(/(PRODUCT DETAILS|DESCRIPTION|&nbsp;)/gi, "");
    if(descriptionFromPDP?.toString().trim()){
        return descriptionFromPDP?.toString().trim()
    }
    else{
        return descriptionFromPDP2?.toString().trim()
    }
})
  let i=0;
    data.forEach(dat=>{
        if((!descriptionFromPDP && data[i].content?.includes('<table'))|| !descriptionFromPDP){
              data = undefined
        }
        else{
             data[i].content = data[i].content.replace(/(?=<table)([\s\S])+(?<=<\/table>)/g, "")
        }
        i++;
    })
    return data
}
```
