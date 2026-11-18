---
name: bryntum-from-extjs
description: >
  Moved to `bryntum-extjs-migration`. Use alongside the `bryntum` skill whenever Ext JS habits
  could leak in — `store.first()`, `component.down()`, `store.get(id)`, `Ext.getCmp`,
  `Ext.create`, `Ext.define`, `xtype`, `dataIndex`, `renderTo`, `Ext.Msg`, `Ext.Ajax`,
  `grid.getStore()`, `getSelectionModel()`, `store.sync()`, or when the user mentions Ext JS,
  ExtJS, Sencha, `Sch.panel.SchedulerGrid` or `Gnt.panel.Gantt`.
metadata:
  tags: bryntum, extjs, sencha, migration, moved
---

# Moved: use `bryntum-extjs-migration`

This skill's content now lives in `bryntum-extjs-migration`. This stub only keeps the old install command
(`npx degit bryntum/skills/bryntum-from-extjs`) and the old raw URL working until the docs guide and the MCP skill list
point at the new skill. Don't add content here.

Load `bryntum-extjs-migration` instead. If it is not installed, fetch:

- Skill: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-extjs-migration/SKILL.md
- Ext → Bryntum idioms (the old content of this skill: near-misses that fail silently such as `store.first()` being a
  getter, `store.find` vs `findRecord`, `down()` vs `widgetMap`, plus the component, store, record, tree, dialog,
  listener and selection maps): https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-extjs-migration/references/idioms.md
