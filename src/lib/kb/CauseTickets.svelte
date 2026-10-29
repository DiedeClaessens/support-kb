<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // `filed`: cause → the ticket opened for it. `timezone` is the one-day moves (desk-notes.json),
  // counted where they're filed and listed again on their own row.
  let { ticket, filed }: { ticket: number; filed: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const moved = $derived(groups.filter((g) => readAs(g, notes.oneDayMove)));
  const link = (url: string | undefined) => url?.replace(/^.*\/issues\//, '#');
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} ·
  <strong>{groups.length + moved.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if filed[g.cause]}<a href={filed[g.cause]}>{link(filed[g.cause])}</a>{:else}none yet{/if}
        </td>
      </tr>
    {/each}
    {#each moved as g (g.cause)}
      <tr class="moved">
        <td><code>timezone</code></td>
        <td>{readAs(g, notes.oneDayMove)}</td>
        <td colspan="2">one-day moves, counted under <code>{g.cause}</code></td>
        <td>
          {#if filed.timezone}<a href={filed.timezone}>{link(filed.timezone)}</a>{:else}none yet{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .moved td { color: var(--muted); }
</style>
