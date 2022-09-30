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
