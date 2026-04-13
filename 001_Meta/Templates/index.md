# Title

```dataviewjs
const PARENT_FIELD = "up";
const ROOT = dv.current().file.path;
// --- normalize parent field ---
function getParent(page) {
  let raw = page[PARENT_FIELD];
  if (!raw) return null;
  if (Array.isArray(raw)) raw = raw[0];
  if (raw?.path) return dv.page(raw.path);
  if (typeof raw === "string") return dv.page(raw);
  return null;
}
let tree = {};
dv.pages()
  .where(p => p[PARENT_FIELD])
  .forEach(p => {
    const parent = getParent(p);
    if (!parent) return;
    const name = parent.file.path;
    tree[name] ??= [];
    tree[name].push(p);
  });
function renderNode(name, level = 0, visited = new Set()) {
  if (visited.has(name)) return [];
  visited.add(name);
  let lines = [];
  const children = tree[name] ?? [];
  children
    .sort(c => c.file.name)
    .forEach(child => {
    const display = child.question ?? child.file.name; // frontmatter field "question"
    const link = `[[${child.file.path}|${display}]]`;
      lines.push(
        "    ".repeat(level) + "- " + link
      );
      lines.push(
        ...renderNode(child.file.path, level + 1, visited)
      );
    });
  return lines;
}
// --- render as markdown ---
const output = "## Index\n" + renderNode(ROOT).join("\n");
dv.el("div", output, { cls: "markdown-rendered" });
```




