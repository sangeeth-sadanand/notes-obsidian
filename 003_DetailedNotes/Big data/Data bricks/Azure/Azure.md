---
tags:
  - Databricks/cloud
up:
  - "[[003_DetailedNotes/Big data/Data bricks/Data bricks|Data bricks]]"
index: 1
type: chapter
---
# Azure

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
	// 2. Create the H3 title inside the row, removing default margins
    let header = dv.container.createEl("h1", { 
        attr: { style: "margin-top: 1em; margin-bottom: 0.5em;" } 
    });
    
    // 2. Build the clickable Obsidian link inside the header
    let link = header.createEl("a", { 
        text: page.file.name+ "🔗", 
        cls: "internal-link" 
    });
    
    // This makes sure Obsidian's hover-preview pop-up still works
    link.setAttribute("data-href", page.file.path); 
    link.setAttribute("href", page.file.path);

    let tFile = app.vault.getAbstractFileByPath(page.file.path);
    if (tFile) {
        let content = await app.vault.read(tFile);
        
        // Regex matches everything inside ```ad-summary ... ```
        let match = content.match(/```ad-summary\s*\n([\s\S]*?)```/i);
        
        if (match && match[1]) {
            // Strip out Admonition config settings (title, collapse, icon, etc.)
            let cleanSummary = match[1]
                .replace(/^title:.*$/gm, "")
                .replace(/^collapse:.*$/gm, "")
                .replace(/^icon:.*$/gm, "").trim();

            // Render with dv.paragraph so Markdown formatting and links work
            dv.paragraph(cleanSummary);
        } else {
            dv.paragraph("*No summary found.*");
        }
    }
}
```

