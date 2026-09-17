---
tags:
  - DataBricks
index: 4
up:
  - "[[001_SummaryNote/Big data|Big data]]"
type: Module
---
# Data bricks

```dataviewjs

let children = dv.pages().where(p => {
    if (!p.up) return false;
    
    let upLinks = dv.array(p.up); 
    return upLinks.some(link => link.path === dv.current().file.path);
});

let sortedNotes = children.array().sort((a, b) => {
	// Grab the index. If missing or null, fallback to 999.
	let indexA = (a.index !== undefined && a.index !== null) ? a.index : 999;
	let indexB = (b.index !== undefined && b.index !== null) ? b.index : 999;
	return indexA - indexB;
});
    
  
for (let page of sortedNotes) {
    dv.paragraph(`![[${page.file.name}]]`); 
}
```