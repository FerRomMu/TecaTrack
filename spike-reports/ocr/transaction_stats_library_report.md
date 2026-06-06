| Criterio | Recharts | @ant-design/charts |
|----------|----------|-------------------|
| React 19 compat | v2.x tiene issues reportados con React 19; v3 (beta) lo soporta pero aún inestable | Compatible con React 19 vía AntV G2 (G2 5.x en adelante) |
| TypeScript | Tipos incluidos (`@types/recharts` no necesario) | Tipos incluidos, bien integrado con el ecosistema TS |
| Line chart + multi-series | Sencillo: `<LineChart>` + `<Line>` por serie | Line config-based, igual de capaz pero API más verbosa |
| Tooltip personalizado | `<Tooltip content={<Custom />} />` — muy limpio en JSX | Configurable vía `tooltip.items` / `tooltip.render` — más declarativo |
| Tamaño del bundle | ~130 KB gzip (solo Recharts + D3 deps) | ~350–500 KB gzip (incluye G2, AntV runtime) |


Como el proyecto ya tiene antd como dependencia central, se decidió agregar @ant-design/charts ya que no introduce un sistema de diseño adicional — reutiliza los tokens de color, tipografía y modo oscuro/claro del tema que ya configuraron en shared/theme.tsx.
