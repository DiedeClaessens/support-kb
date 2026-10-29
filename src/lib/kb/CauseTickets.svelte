<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // `causeOf` (causes.ts) ties each report to a cause. It has two: parser and hint. That is the
  // desk's own view. `filed`: cause → the ticket opened for it and how it ended.
  let { ticket, filed }: { ticket: number; filed: Record<string, { url: string; status: string }> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf).filter((g) => g.cause !== 'unplaced'));
  const link = (url: string) => url.replace(/^.*\/issues\//, '#');
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} ·
  <strong>{groups.length}</strong> causes (<code>causeOf</code>, causes.ts)
</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>how it ended</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if filed[g.cause]}<a href={filed[g.cause].url}>{link(filed[g.cause].url)}</a>{:else}none{/if}</td>
        <td>{filed[g.cause]?.status ?? 'open'}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
