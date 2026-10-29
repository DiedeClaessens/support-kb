<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // `filed`: problem → the ticket opened for it and how it ended. `timezone` is the one-day moves
  // (desk-notes.json): the desk files them under `parser`; here they are a problem of their own.
  let { ticket, filed }: { ticket: number; filed: Record<string, { url: string; status: string }> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const moved = $derived(groups.reduce((n, g) => n + readAs(g, notes.oneDayMove), 0));
  const rows = $derived([
    ...groups.map((g) => ({ problem: g.cause as string, reports: g.reports.length - readAs(g, notes.oneDayMove), tenants: g.tenants, seen: `${g.firstSeen} → ${g.lastSeen}` })),
    { problem: 'timezone', reports: moved, tenants: new Set(merged.filter((r) => notes.oneDayMove.includes(r.id)).map((r) => r.tenant)).size, seen: 'one-day moves, no swap' },
  ]);
  const link = (url: string) => url.replace(/^.*\/issues\//, '#');
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} ·
  <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>how it ended</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code></td>
        <td>{r.reports}</td>
        <td>{r.tenants}</td>
        <td>{r.seen}</td>
        <td>{#if filed[r.problem]}<a href={filed[r.problem].url}>{link(filed[r.problem].url)}</a>{:else}none yet{/if}</td>
        <td>{filed[r.problem]?.status ?? 'open'}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
