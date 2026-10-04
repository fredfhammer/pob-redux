<script lang="ts">
  import type { Snippet } from "svelte";
  import type { NodeOverride } from "$lib/engine.svelte";
  import type { TNode } from "$lib/tree/model";
  import { m } from "$lib/paraglide/messages";

  type Ref = { label: string; text: string; rarity?: string | null; note?: string | null };
  let {
    node,
    override,
    notCalc = [],
    allocated = false,
    refs = [],
    fixed = false,
    left,
    top,
    el = $bindable(null),
    children,
  }: {
    node: TNode;
    override?: NodeOverride;
    notCalc?: string[];
    allocated?: boolean;
    refs?: Ref[];
    fixed?: boolean;
    left: number;
    top: number;
    el?: HTMLDivElement | null;
    children?: Snippet;
  } = $props();

  const nc = $derived(new Set(notCalc));
</script>

<div class="tip" class:fixed bind:this={el} style:left={`${left}px`} style:top={`${top}px`}>
  <div class="tip-head">
    <span class="tip-name" class:key={node.kind === "keystone"} class:notable={node.kind === "notable"}>{override?.name ?? node.name}</span>
    <span class="label">{node.asc ?? node.kind}</span>
  </div>
  {#each refs as r}
    <div class="tip-stat">
      <span class="dim">{r.label}</span>
      <span class="rarity" data-rarity={r.rarity ?? ""}>{r.text}</span>{#if r.note}<span class="dim"> ({r.note})</span>{/if}
    </div>
  {/each}
  {#each override?.stats?.length ? override.stats : node.stats as s}
    {@const bad = nc.size > 0 && s.split("\n").some((l) => nc.has(l))}
    <div class="tip-stat" class:nc={bad}>{s}{#if bad}<span class="ncnote">{` ${m.not_calculated()}`}</span>{/if}</div>
  {/each}
  {#if node.masteryEffects && !allocated}
    {#each node.masteryEffects as e (e.effect)}
      <div class="tip-stat dim">{e.stats.join(" / ")}</div>
    {/each}
  {/if}
  {#if node.flavour}
    <div class="tip-flav">{node.flavour}</div>
  {/if}
  {@render children?.()}
</div>

<style>
  .tip {
    position: absolute;
    width: max-content;
    min-width: min(320px, calc(100% - 16px));
    max-width: min(540px, calc(100% - 16px));
    max-height: calc(100% - 16px);
    overflow: hidden;
    padding: 10px 12px;
    background: color-mix(in srgb, var(--bg-1) 94%, transparent);
    border: 1px solid var(--line-1);
    border-radius: var(--r-2);
    box-shadow: var(--shadow-pop);
    pointer-events: none;
    font-size: var(--fs-sm);
  }
  .tip.fixed {
    position: fixed;
    z-index: 10;
  }
  .tip-head {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    gap: 8px;
    margin-bottom: 6px;
  }
  .tip-name {
    font-weight: 600;
    color: var(--fg-0);
  }
  .tip-name.key {
    color: var(--c-rare);
  }
  .tip-name.notable {
    color: var(--c-currency);
  }
  .tip-stat {
    color: var(--c-magic);
    line-height: 1.35;
    white-space: pre-line;
  }
  .tip-stat.nc {
    color: var(--bad);
  }
  .ncnote {
    color: var(--fg-3);
  }
  .tip-stat .rarity {
    color: var(--fg-0);
  }
  .tip-stat .rarity[data-rarity="UNIQUE"] {
    color: var(--c-unique);
  }
  .tip-stat .rarity[data-rarity="RARE"] {
    color: var(--c-rare);
  }
  .tip-stat .rarity[data-rarity="MAGIC"] {
    color: var(--c-magic);
  }
  .tip-flav {
    margin-top: 6px;
    color: var(--c-unique);
    font-style: italic;
    font-size: var(--fs-xs);
  }
</style>
