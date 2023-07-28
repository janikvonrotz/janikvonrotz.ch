---
title: "Convert Obsidian canvas to SVG"
slug: convert-obsidian-canvas-to-svg
date: 2023-06-07T09:47:45+02:00
categories:
 - Knowledge
tags:
 - obisidan
 - svg
 - canvas
images:
 - /images/obsidian-logo-old.png
---

I am using [Obsidian](https://obsidian.md/) for note taking and writing documentation. With [Canvas-Feature](https://obsidian.md/canvas) you can create simple visualizations and link notes from the vault. Here is an example:

![](/images/obsidian-canvas-editor.png)

As simple as it seems, there was not builtin-way to export the visualization and f.g. publish it on a website. Luckily, the canvas file is a simple JSON document and therefore can be processed pretty easy. I build a render Method in JavaScript that takes the canvas content and returns it as a SVG image.

<!--more-->

Here is what it a canvas file looks like:

**Obsidian.canvas**

```json
{
	"nodes":[
		{"id":"c2b83b58ff104957","x":-150,"y":-200,"width":250,"height":60,"color":"1","type":"text","text":"Create canvas in Obsidian"},
		{"id":"28b7d169be420aea","x":-200,"y":-80,"width":350,"height":240,"color":"#803dd1","type":"file","file":"Obsidian.md"},
		{"type":"text","text":"Export as SVG","id":"3bbb4a8b9a8a726b","x":240,"y":10,"width":250,"height":60,"color":"5"}
	],
	"edges":[
		{"id":"9a41a93da6186142","fromNode":"c2b83b58ff104957","fromSide":"bottom","toNode":"28b7d169be420aea","toSide":"top"},
		{"id":"14fcc8b5466e0f72","fromNode":"28b7d169be420aea","fromSide":"right","toNode":"3bbb4a8b9a8a726b","toSide":"left"}
	]
}
```

Nodes are the rectangles in the canvas editor and edges are the arrows connecting them.

With the help of JavaScript I was able to generate this SVG:

**Obsidian.svg**

![](/images/Obsidian.svg)

The script has several methods:

* **mapColor**: Maps the Obsidian color id to the actual hex color
* **renderNode**: Given a canvas node it renders a SVG rectangle and either show a text, link markdown file or embeding an image
* **renderGroup**: Renders a SVG rectrange from a canvas group
* **renderEdge**: Given a canvas edge it renders a SVG line with a marker
* **convertCanvasToSVG**: This is the main function that process the nodes and edges and calculates the position for the elements

The main challenge was to calculate the actual positions of the arrows. As you can see in the canvas document the edges do not have a fixed position.

```js
function mapColor(color) {
    colors = {
        0: '#7e7e7e',
        1: '#aa363d',
        2: '#a56c3a',
        3: '#aba960',
        4: '#199e5c',
        5: '#249391',
        6: '#795fac'
    }
    let appliedColor = colors[0]
    
    if (color && (0 < color.length < 2)) {
        appliedColor = colors[color]
    }
    if (color && (1 < color.length)) {
        appliedColor = color
    }
    return appliedColor
}
```

```js
function renderNode(node) {
    const strockWidth = 4
    const fontWeight = 'bold'
    const fontFamily = 'Roboto, Oxygen, Ubuntu, Cantarell, sans-serif'

    let textOffsetX = 15
    let textOffsetY = 0
    let fontColor = '#2c2d2c'
    let fontSize = 15
    let content = ''

    // Render default text

    if (node['text']) {
        content = `
        <style>
            p {
                font-family: ${fontFamily};
                font-size: ${fontSize}px;
                color: ${fontColor};
            }
        </style>
        <foreignObject x="${node['x'] + textOffsetX}" y="${node['y'] + textOffsetY}" width="${node['width'] - textOffsetX*2}" height="${node['height'] - textOffsetY*2}">
        <p xmlns="http://www.w3.org/1999/xhtml" class="${node['id']}">${node['text']}</p>
        </foreignObject>
        `
    }

    // Render multiline text

    if (node['text'] && node['text'].split('\n').length > 1) {
        let spans = ''
        for (const line of node['text'].split('\n')) {
            spans += `<tspan x="${node['x'] + textOffsetX}" dy="${fontSize + 3}">${line}</tspan>`
        }
        textOffsetY = 10
        content = `<text x="${node['x'] + textOffsetX}" y="${node['y'] + textOffsetY}" font-family="${fontFamily}" fill="${fontColor}">${spans}</text>`
    }

    // Render linked markdown file

    if (node['file'] && node['file'].endsWith('.md')) {
        title = node['file'].replace('.md', '')
        text = `<a href="/${title.toLowerCase()}.html">${title}</a>`
        fontColor = '#9a7fee'
        fontSize = 28
        textOffsetX = 30
        textOffsetY = 45
        content = `<text x="${node['x'] + textOffsetX}" y="${node['y'] + textOffsetY}" font-family="${fontFamily}" font-size="${fontSize}" font-weight="${fontWeight}" fill="${fontColor}">${text}</text>`
    }
    
    // Render image

    if (node['file'] && !node['file'].endsWith('.md')) {
        filePath = node['file']

        const base64_content = fs.readFileSync(filePath, "base64")
        extension = path.extname(filePath).replace('.', '')

        content = `<image href="${`data:image/{extension};base64,${base64_content}`}" x="${node['x']}" y="${node['y']}" width="${node['width']}" height="${node['height']}" clip-path="inset(0% round 15px)" />`
        fontColor = '#9a7fee'
    }

    return `
    <rect x="${node['x']}" y="${node['y']}" width="${node['width']}" height="${node['height']}" rx="15" stroke="${mapColor(node['color'])}" stroke-width="${strockWidth}" fill="none"/>
    ${content}
    `
}
```

```js
function renderGroup(group) {
    const strockWidth = 4
    const fontWeight = 'bold'
    const fontFamily = 'Roboto, Oxygen, Ubuntu, Cantarell, sans-serif'

    let textOffsetX = 15
    let textOffsetY = -15
    let fontColor = '#2c2d2c'
    let fillColor = '#fbfbfb'
    let text = group['label']
    let fontSize = 24

    return `
    <rect x="${group['x']}" y="${group['y']}" width="${group['width']}" height="${group['height']}" rx="30" stroke="${mapColor(group['color'])}" stroke-width="${strockWidth}" fill="${fillColor}"/>
    <text x="${group['x'] + textOffsetX}" y="${group['y'] + textOffsetY}" font-family="${fontFamily}" font-size="${fontSize}" font-weight="${fontWeight}" fill="${fontColor}">${text}</text>
    `
}
```

```js
function renderEdge(edge) {
    const id = edge['id']
    const strockWidth = 5
    const color = mapColor(edge['color'])
    const fromSide = edge['fromSide']
    const toSide = edge['toSide']
    const fontFamily = 'Roboto, Oxygen, Ubuntu, Cantarell, sans-serif'
    const fontColor = '#2c2d2c'

    let marker = `marker-end="url(#arrow-end-${id})"`
    let fromOffset = 1
    let toOffset = 11
    let fromX = edge['fromX'] 
    let fromY = edge['fromY']
    let toX = edge['toX']
    let toY = edge['toY']
    let label = ''

    // Set arrow marker

    if(edge['fromEnd'] === 'arrow') {
        marker = `marker-end="url(#arrow-end-${id})" marker-start="url(#arrow-start-${id})"`
        fromOffset = 11
    }
    if(edge['toEnd'] === 'none') {
        marker = ''
        toOffset = 1
    }

    // Calculate position with offset

    if (fromSide === 'right') {
        fromX += fromOffset
    }
    if (fromSide === 'bottom') {
        fromY += fromOffset
    }
    if (fromSide === 'left') {
        fromX -= fromOffset
    }
    if (fromSide === 'top') {
        fromY -= fromOffset
    }
    if (toSide === 'right') {
        toX += toOffset
    }
    if (toSide === 'bottom') {
        toY += toOffset
    }
    if (toSide === 'left') {
        toX -= toOffset
    }
    if (toSide === 'top') {
        toY -= toOffset
    }

    // Add label if is set

    if(edge['label']) {
        
        // Calculate position with offset
        let labelLength = edge['label'].length*4
        let labelX = fromX - labelLength
        let labelY = fromY

        if (toX > fromX) {
            labelX += Math.abs((fromX-toX)/2)
        }
        if (toY > fromY) {
            labelY += Math.abs((fromY-toY)/2)
        }
        if (toX < fromX) {
            labelX -= Math.abs((toX-fromY)/2)
        }
        if (toY < fromY) {
            labelY -= Math.abs((toY-fromY)/2)
        }

        label = content = `<text x="${labelX}" y="${labelY}" font-family="${fontFamily}" fill="${fontColor}">${edge['label']}</text>`
    }

    return `
    <marker xmlns="http://www.w3.org/2000/svg" id="arrow-end-${id}" viewBox="0 0 10 10" refX="1" refY="5" fill="${color}" markerUnits="strokeWidth" markerWidth="3" markerHeight="3" orient="auto">
        <path d="M 0 0 L 7 5 L 0 10 z"/>
    </marker>
    <marker xmlns="http://www.w3.org/2000/svg" id="arrow-start-${id}" viewBox="-10 -10 10 10" refX="-1" refY="-5" fill="${color}" markerUnits="strokeWidth" markerWidth="3" markerHeight="3" orient="auto">
        <path d="M 0 0 L -7 -5 L -0 -10 z"/>
    </marker>
    <line x1="${fromX}" y1="${fromY}" x2="${toX}" y2="${toY}" stroke="${color}" stroke-width="${strockWidth}" ${marker} />
    ${label}
    `
}
```

```js
function convertCanvasToSVG(content) {

    nodes = content['nodes']
    edges = content['edges']

    let svg = ""
    svg += '<?xml version="1.0" encoding="UTF-8" standalone="no"?>\n'
    svg += '<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">\n'    

    // Calculate view box position

    let minX = 0
    let minY = 0

    for (const node of nodes) {
        nodeX = node['x']
        nodeY = node['y']
        nodeWith = node['width']
        nodeHeight = node['height']

        if (nodeX < minX) {
            minX = nodeX
        }
        if (nodeY < minY) {
            minY = nodeY
        }
    }

    // Caclulate view box size

    let width = 0
    let height = 0

    for (const node of nodes) {
        nodeX = node['x']
        nodeY = node['y']
        nodeWith = node['width']
        nodeHeight = node['height']

        nodeMaxX = Math.abs(nodeX - minX) + nodeWith
        if (width < nodeMaxX) {
            width = nodeMaxX
        }
        nodeMaxY = Math.abs(nodeY - minY) + nodeHeight
        if (height < nodeMaxY) {
            height = nodeMaxY
        }
    }

    // Add view box

    const spacing = 50
    
    svg += `<svg viewBox="${minX-spacing} ${minY-spacing} ${width+spacing*2} ${height+spacing*2}" xmlns="http://www.w3.org/2000/svg">\n`

	// Render group as rect

    for (const group of nodes.filter(node => (node['type'] === 'group'))) {
        svg += renderGroup(group)
    }
    
    for (const edge of edges) {
        const fromSide = edge['fromSide']
        const toSide = edge['toSide']
        let fromX = 0
        let fromY = 0
        let toX = 0
        let toY = 0

        // Get start and target nodes

        fromNode = nodes.filter(node => (node['id'] === edge['fromNode']))[0]
        toNode = nodes.filter(node => (node['id'] === edge['toNode']))[0]

        // Calculate x and y position of arrow start

        if (fromSide === 'right') {
            fromX = fromNode['x'] + fromNode['width']
            fromY = fromNode['y'] + fromNode['height'] / 2
        }
        if (fromSide === 'bottom') {
            fromX = fromNode['x'] + fromNode['width'] / 2 
            fromY = fromNode['y'] + fromNode['height']
        }
        if (fromSide === 'left') {
            fromX = fromNode['x']
            fromY = fromNode['y'] + fromNode['height'] / 2
        }
        if (fromSide === 'top') {
            fromX = fromNode['x'] + fromNode['width'] / 2
            fromY = fromNode['y']
        }
        edge['fromX'] = fromX
        edge['fromY'] = fromY

        // Calculate x and y position of arrow target        

        if (toSide === 'right') {
            toX = toNode['x'] + toNode['width']
            toY = toNode['y'] + toNode['height'] / 2
        }
        if (toSide === 'bottom') {
            toX = toNode['x'] + toNode['width'] / 2 
            toY = toNode['y'] + toNode['height']
        }
        if (toSide === 'left') {
            toX = toNode['x']
            toY = toNode['y'] + toNode['height'] / 2
        }
        if (toSide === 'top') {
            toX = toNode['x'] + toNode['width'] / 2
            toY = toNode['y']
        }
        edge['toX'] = toX
        edge['toY'] = toY

        svg += renderEdge(edge)
    }

    // Render nodes as rect

    for (const node of nodes.filter(node => (['text', 'file'].includes(node['type'])))) {
        svg += renderNode(node)
    }

    svg += '</svg>'

    return svg
}
```

I am pretty sure the calculation can be done more efficiently. Nonetheless, it was a lot of fun to wrap my head around this problem.