<script lang="ts">
  import { crossfade } from 'svelte/transition';
  import { flip } from 'svelte/animate';

  import { slide, crossfadeFade } from '../../shared/transitions';

  import { useNetworkInfo } from '../client/network';

  export let id: string;
  export let authToken: string | undefined;

  const animationOptions = { duration: (d: number) => 30 * Math.sqrt(d) };
  const [send, receive] = crossfade({ fallback: crossfadeFade });

  $: networkInfoRes = useNetworkInfo(id, authToken);
  $: networkInfoStatus = $networkInfoRes.status;
  $: networkInfo = $networkInfoRes.data?.data;
</script>

<div class="accordion-content" transition:slide|local>
  {#if networkInfoStatus === 'loading'}
    Loading...
  {:else if networkInfo === undefined}
    <p class="tag is-warning">Unknown</p>
  {:else}
    <h4 class="is-size-6">Network Managed Routes</h4>
    {#if networkInfo.routes.length === 0}
      <p class="tag is-warning">
        {#if networkInfo.status !== 'OK'}
          Unknown
        {:else}
          None
        {/if}
      </p>
    {:else}
      {#each networkInfo.routes as { target, via } (target)}
        <p
          class="route"
          in:receive|local={{ key: target }}
          out:send|local={{ key: target }}
          animate:flip={animationOptions}
        >
          <span class="tag ip-subnet">{target}</span>
          {#if via !== null}
            via <span class="tag ip-subnet">{via}</span>
          {:else}
            (local)
          {/if}
        </p>
      {/each}
    {/if}
    <h4 class="is-size-6">Virtual Network Device</h4>
    <p>
      Name:
      {#if networkInfo.portDeviceName.length === 0}
        <span class="tag is-warning">None</span>
      {:else}
        {networkInfo.portDeviceName}
      {/if}
    </p>
    <p>
      MAC Address:
      {#if networkInfo.mac.length === 0}
        <span class="tag is-warning">None</span>
      {:else}
        <span class="tag mac-address">{networkInfo.mac}</span>
      {/if}
    </p>
    <p>MTU: {networkInfo.mtu}</p>
    <h4 class="is-size-6">Network Configuration</h4>
    <p>
      Broadcast:
      {#if networkInfo.broadcastEnabled}
        Enabled
      {:else}
        Disabled
      {/if}
    </p>
    <p>Revision ID: {networkInfo.netconfRevision}</p>
  {/if}
</div>

<style>
  .accordion-content p.route:not(:last-child) {
    margin-bottom: 0.25em;
  }
</style>
