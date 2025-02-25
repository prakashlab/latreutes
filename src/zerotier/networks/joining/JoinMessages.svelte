<script lang="ts">
  import { useTxtResolver } from '../../../shared/dns';
  import { slide } from '../../../shared/transitions';

  import type { Status, Type } from '../../client/network';
  import { checkNetworkDomainName } from '../../client/network';

  import AccessDenied from './messages/AccessDenied.svelte';
  import NotFound from './messages/NotFound.svelte';
  import PortError from './messages/PortError.svelte';
  import OldVersion from './messages/OldVersion.svelte';
  import MissingName from './messages/MissingName.svelte';
  import NoDomainName from './messages/NoDomainName.svelte';
  import DomainName from './messages/DomainName.svelte';
  import PublicNetwork from './messages/PublicNetwork.svelte';

  export let id: string;
  export let name: string;
  export let status: Status;
  export let type: Type;
  export let expectedName: string | undefined;

  $: txtRecordsRes = useTxtResolver(name);
  $: hasConfirmedDomainName = checkNetworkDomainName(
    name,
    id,
    // eslint-disable-next-line @typescript-eslint/no-unsafe-argument
    $txtRecordsRes.data,
    // eslint-disable-next-line @typescript-eslint/no-unsafe-argument
    $txtRecordsRes.status,
  );
</script>

{#if status === 'ACCESS_DENIED'}
  <article class="message is-info" transition:slide|local>
    <AccessDenied />
  </article>
{:else if status === 'NOT_FOUND'}
  <article class="message is-danger" transition:slide|local>
    <NotFound />
  </article>
{:else if status === 'PORT_ERROR'}
  <article class="message is-danger" transition:slide|local>
    <PortError />
  </article>
{:else if status === 'CLIENT_TOO_OLD'}
  <article class="message is-danger" transition:slide|local>
    <OldVersion />
  </article>
{:else if status === 'OK' && !name}
  <article class="message is-warning" transition:slide|local>
    <MissingName />
  </article>
{:else if status === 'OK' && expectedName === undefined && hasConfirmedDomainName}
  <article class="message is-info" transition:slide|local>
    <DomainName {name} nameExpected={false} />
  </article>
{:else if status === 'OK' && expectedName === undefined}
  <article class="message is-warning" transition:slide|local>
    <NoDomainName {name} nameExpected={false} />
  </article>
{:else if status === 'OK' && name !== expectedName && hasConfirmedDomainName}
  <article class="message is-warning" transition:slide|local>
    <DomainName {name} nameExpected={true} />
  </article>
{:else if status === 'OK' && name !== expectedName}
  <article class="message is-warning" transition:slide|local>
    <NoDomainName {name} nameExpected={true} />
  </article>
{/if}
{#if type === 'PUBLIC'}
  <article class="message is-warning" transition:slide|local>
    <PublicNetwork />
  </article>
{/if}
