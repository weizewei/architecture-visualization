# Interactive Canvas DAG + openFile

ContextWeave-style pattern: SVG graph + click → IDE source.

## Jump API

```tsx
const dispatch = useCanvasAction();

dispatch({
  type: "openFile",
  path: "/Users/you/Code/WebDev/qianxun/facade/es/SyncApi.php", // absolute preferred
  selection: {
    startLineNumber: 34,
    startColumn: 1,
    endLineNumber: 34,
    endColumn: 1,
  },
});
```

- Omit `selection` when line is unknown.
- `path` may be workspace-relative; absolute is safer with multi-root workspaces.

## Template

```tsx
import {
  computeDAGLayout,
  Stack,
  Row,
  H1,
  H2,
  Text,
  Table,
  Button,
  Callout,
  useHostTheme,
  useCanvasAction,
  useCanvasState,
  type DAGLayoutEdge,
} from "cursor/canvas";

type Kind = "entry" | "service" | "event" | "consumer" | "store" | "job" | "other";

type ArchNode = {
  id: string;
  label: string;
  detail?: string;
  path?: string;
  line?: number;
  kind?: Kind;
};

const NODES: ArchNode[] = [
  {
    id: "trigger",
    label: "triggerInfoSync",
    detail: "MetaService",
    kind: "entry",
    path: "/Users/you/Code/WebDev/qianxun/facade/meta/service/MetaService.php",
    line: 23,
  },
  {
    id: "sync",
    label: "Es SyncApi",
    detail: "updateDocument",
    kind: "store",
    path: "/Users/you/Code/WebDev/qianxun/facade/es/SyncApi.php",
    line: 34,
  },
];

const EDGES = [
  { from: "trigger", to: "sync", label: "…via consumer" },
] as const;

function openSource(
  dispatch: (a: { type: "openFile"; path: string; selection?: object }) => void,
  node: ArchNode,
) {
  if (!node.path) return;
  if (node.line && node.line > 0) {
    dispatch({
      type: "openFile",
      path: node.path,
      selection: {
        startLineNumber: node.line,
        startColumn: 1,
        endLineNumber: node.line,
        endColumn: 1,
      },
    });
  } else {
    dispatch({ type: "openFile", path: node.path });
  }
}

export default function ArchitectureCanvas() {
  const theme = useHostTheme();
  const dispatch = useCanvasAction();
  const [selectedId, setSelectedId] = useCanvasState<string | null>("selected-node", null);

  const nodeW = 168;
  const nodeH = 56;
  const layout = computeDAGLayout({
    nodes: NODES.map((n) => ({ id: n.id })),
    edges: EDGES.map((e) => ({ from: e.from, to: e.to })),
    direction: "horizontal",
    nodeWidth: nodeW,
    nodeHeight: nodeH,
    rankGap: 72,
    nodeGap: 28,
    padding: 16,
  });

  const byId = Object.fromEntries(NODES.map((n) => [n.id, n]));
  const selected = selectedId ? byId[selectedId] : undefined;
  const edgeLabel = Object.fromEntries(EDGES.map((e) => [`${e.from}->${e.to}`, e.label]));

  return (
    <Stack gap={20}>
      <Stack gap={6}>
        <H1>Example flow</H1>
        <Text tone="secondary" size="small">
          Source: codebase · click a node to open source
        </Text>
      </Stack>

      <svg width={layout.width} height={layout.height} style={{ maxWidth: "100%" }}>
        <defs>
          <marker id="arrow" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
            <path d="M0,0 L6,3 L0,6 Z" fill={theme.stroke.primary} />
          </marker>
        </defs>

        {layout.edges.map((e: DAGLayoutEdge, i: number) => {
          const midX = (e.sourceX + e.targetX) / 2;
          const midY = (e.sourceY + e.targetY) / 2;
          const key = `${e.from}->${e.to}`;
          return (
            <g key={i}>
              <line
                x1={e.sourceX}
                y1={e.sourceY}
                x2={e.targetX}
                y2={e.targetY}
                stroke={theme.stroke.primary}
                strokeWidth={1.5}
                strokeDasharray={e.isBackEdge ? "4 4" : undefined}
                markerEnd="url(#arrow)"
              />
              {edgeLabel[key] ? (
                <text x={midX} y={midY - 6} textAnchor="middle" fill={theme.text.secondary} fontSize={11}>
                  {edgeLabel[key]}
                </text>
              ) : null}
            </g>
          );
        })}

        {layout.nodes.map((n) => {
          const meta = byId[n.id];
          const jumpable = Boolean(meta?.path);
          const isSelected = selectedId === n.id;
          const stroke = isSelected
            ? theme.accent.primary
            : meta?.kind === "store"
              ? theme.accent.primary
              : theme.stroke.primary;

          return (
            <g
              key={n.id}
              transform={`translate(${n.x}, ${n.y})`}
              style={{ cursor: jumpable ? "pointer" : "default" }}
              onClick={() => {
                setSelectedId(n.id);
                if (meta) openSource(dispatch, meta);
              }}
            >
              <title>{meta?.path ? `${meta.path}${meta.line ? `:${meta.line}` : ""}` : meta?.label}</title>
              <rect
                width={nodeW}
                height={nodeH}
                rx={6}
                fill={isSelected ? theme.fill.tertiary : theme.bg.elevated}
                stroke={stroke}
                strokeWidth={isSelected ? 2 : 1.5}
              />
              <text x={nodeW / 2} y={22} textAnchor="middle" fill={theme.text.primary} fontSize={13} fontWeight={600}>
                {meta?.label ?? n.id}
              </text>
              {meta?.detail ? (
                <text x={nodeW / 2} y={40} textAnchor="middle" fill={theme.text.secondary} fontSize={11}>
                  {meta.detail}
                </text>
              ) : null}
            </g>
          );
        })}
      </svg>

      {selected ? (
        <Callout tone="info" title={selected.label}>
          <Stack gap={8}>
            <Text size="small">
              {selected.path
                ? `${selected.path}${selected.line ? `:${selected.line}` : ""}`
                : "No source path for this node"}
            </Text>
            {selected.path ? (
              <Row gap={8}>
                <Button
                  variant="primary"
                  onClick={() => openSource(dispatch, selected)}
                >
                  Open in editor
                </Button>
              </Row>
            ) : null}
          </Stack>
        </Callout>
      ) : (
        <Text tone="secondary" size="small">
          Select a node to jump to its source file.
        </Text>
      )}

      <Stack gap={8}>
        <H2>Nodes</H2>
        <Table
          headers={["Node", "Role", "Path"]}
          rows={NODES.map((n) => [
            n.label,
            n.detail ?? "",
            n.path ? `${n.path}${n.line ? `:${n.line}` : ""}` : "—",
          ])}
          // If Table supports row onClick in current SDK, wire openSource there too.
          // Otherwise keep Open via SVG / Callout Button only.
        />
      </Stack>
    </Stack>
  );
}
```

## Checklist before shipping

- [ ] Jumpable nodes have real `path` (absolute when multi-root)
- [ ] `line` set when the symbol line is known
- [ ] Click node opens file (and selects line when provided)
- [ ] Selected state + "Open in editor" button
- [ ] Caption mentions click-to-open
- [ ] Theme tokens only; no gradients / emojis / box-shadow

## Notes

- Confirm `Callout` / `Button` / `Table` props in `~/.cursor/skills-cursor/canvas/sdk/ui-primitives.d.ts` if typecheck fails.
- If `openFile` selection shape drifts, keep `path`-only jump as fallback.
