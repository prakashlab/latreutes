<script lang="ts">
  import { onMount } from 'svelte';
  import { crossfade } from 'svelte/transition';
  import { flip } from 'svelte/animate';

  import { crossfadeFade } from '../../shared/transitions';

  import { useReverseResolver } from '../../shared/dns';

  import type { PathInfo } from '../client/peer';

  export let path: PathInfo;

  let time = new Date().getTime();
  let lastSend = 0;
  let lastReceive = 0;

  const animationOptions = { duration: (d: number) => 30 * Math.sqrt(d) };
  const [send, receive] = crossfade({ fallback: crossfadeFade });

  $: ipAddress = path.address.split('/')[0];
  $: port = path.address.split('/')[1];
  $: isIPv6 = ipAddress.includes(':');
  $: reverseRecordsRes = useReverseResolver(ipAddress);
  $: hasReverseRecords =
    $reverseRecordsRes.status === 'success' &&
    $reverseRecordsRes.data !== undefined;
  $: if (path.lastSend !== 0) {
    lastSend = path.lastSend;
  }
  $: if (path.lastReceive !== 0) {
    lastReceive = path.lastReceive;
  }

  onMount(() => {
    const interval = setInterval(() => {
      time = new Date().getTime();
    }, 300);

    return () => {
      clearInterval(interval);
    };
  });
</script>

<div class="tags">
  <span class="tag socket">
    {#if isIPv6}
      [{ipAddress}]:{port}
    {:else}
      {ipAddress}:{port}
    {/if}
  </span>
  {#if !path.active}
    <span class="tag is-warning">Inactive</span>
  {/if}
  {#if path.expired}
    <span class="tag is-danger">Expired</span>
  {/if}
  {#if path.preferred}
    <span class="tag is-info">Preferred</span>
  {/if}
</div>
{#if hasReverseRecords && $reverseRecordsRes.data !== undefined && $reverseRecordsRes.data.length > 0}
  <div class="tags">
    {#each $reverseRecordsRes.data as domainName (domainName)}
      <span
        class="tag domain-name"
        in:receive|local={{ key: domainName }}
        out:send|local={{ key: domainName }}
        animate:flip={animationOptions}
      >
        {domainName.replace(/.$/, '')}
      </span>
    {/each}
  </div>
{/if}
<p>
  Last sent
  {lastSend === 0
    ? 'an unknown time'
    : ((time - lastSend) / 1000).toFixed(0) + ' s'}
  ago.
  <br />
  Last received
  {lastReceive === 0
    ? 'an unknown time'
    : ((time - lastReceive) / 1000).toFixed(0) + ' s'}
  ago.
</p>

<style>
  .tags:not(:last-child) {
    margin-bottom: 0;
  }
  p:not(:last-child) {
    margin-bottom: 0;
  }
</style>
