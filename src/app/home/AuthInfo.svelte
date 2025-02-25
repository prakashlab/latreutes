<script lang="ts">
  import { useQueryClient } from '@sveltestack/svelte-query';

  import {
    AUTHTOKEN_FILENAME,
    useConfigDirMaker,
    useConfigPath,
    useAuthToken,
    invalidateAuthTokenCache,
  } from '../../shared/config';
  import { usePlatformType } from '../../shared/os';
  import { useShellOpener, useSuCopier } from '../../shared/shell';
  import { slide } from '../../shared/transitions';

  import { getZtOneConfigPath } from '../../zerotier/config';
  import {
    SERVICE_PORT_ZT,
    invalidateCache,
  } from '../../zerotier/client/service';
  import { useNodeInfo } from '../../zerotier/client/node';

  export let authToken: string | undefined;

  const shellOpener = useShellOpener();
  const suCopier = useSuCopier(invalidateCaches);
  const queryClient = useQueryClient();
  const platformTypeRes = usePlatformType();
  const configDirMaker = useConfigDirMaker();
  const configPathRes = useConfigPath();

  let configPath: string | undefined;
  $: configPath = $configPathRes.data;
  let ztOneConfigPath: string;
  $: ztOneConfigPath =
    $platformTypeRes.data === undefined
      ? ''
      : getZtOneConfigPath($platformTypeRes.data);
  $: authTokenPath = `${configPath}${AUTHTOKEN_FILENAME}`;
  $: authTokenRes = useAuthToken(configPath);
  $: authTokenStatus = $authTokenRes.status;
  $: nodeInfoRes = useNodeInfo(authToken);
  $: nodeInfoStatus = $nodeInfoRes.status;

  function invalidateCaches() {
    invalidateAuthTokenCache(queryClient);
    invalidateCache(queryClient);
  }
  function openZtConfigPath() {
    $shellOpener.mutate(ztOneConfigPath);
  }
  function openConfigPath() {
    $configDirMaker.mutate();
    if (configPath !== undefined) {
      $shellOpener.mutate(configPath);
    }
  }
  function copyAuthTokenFile() {
    $configDirMaker.mutate();
    $suCopier.mutate({
      source: `${ztOneConfigPath}${AUTHTOKEN_FILENAME}`,
      dest: `${configPath}${AUTHTOKEN_FILENAME}`,
    });
  }
</script>

<div class="card info-card" in:slide|local>
  <div class="content card-content">
    <h2>Set Up ZeroTier Access</h2>
    {#if configPath === undefined}
      <p in:slide|local>
        This program is unable to determine where the <code
          >authtoken.secret</code
        >
        file should be found! This error is not supposed to happen.
      </p>
    {:else if authTokenStatus === 'loading'}
      {#if !authTokenPath}
        <p in:slide|local>Loading ZeroTier auth token...</p>
      {:else}
        <p in:slide|local>Loading ZeroTier auth token from: {authTokenPath}</p>
      {/if}
    {:else if authTokenStatus === 'error'}
      <div class="content" in:slide|local>
        <p>
          In order for this program to talk to the ZeroTier service running on
          your computer, you&apos;ll need to make a copy of ZeroTier&apos;s
          <code>authtoken.secret</code> file so that this program can access the
          copy. You&apos;ll need administrator permissions on this device in order
          to open or copy the file.
        </p>
        <p>
          You should be able to find the <code>authtoken.secret</code> file at
          <button
            role="link"
            class="button is-ghost is-ahref"
            on:click={openZtConfigPath}
          >
            {ztOneConfigPath}{AUTHTOKEN_FILENAME}
          </button>. You&apos;ll need to copy that file into
          <button
            role="link"
            class="button is-ghost is-ahref"
            on:click={openConfigPath}
          >
            {configPath}
          </button>
          and make sure you can open the copied file with a text editor.
        </p>
        {#if $platformTypeRes.data === 'Linux'}
          <p>
            The easiest way to copy the file so that this program has the
            correct permissions to read the copied file is to run the following
            command in the terminal:
            <br />
            <code class="is-block">
              sudo cat {ztOneConfigPath}{AUTHTOKEN_FILENAME} &gt; {configPath}{AUTHTOKEN_FILENAME}
            </code>
          </p>
          <p>
            You can run that command simply by pressing "Copy File" button
            below; it should prompt you to enter your password so that it can
            read the file with administrator permissions in order to make a copy
            of it. If it doesn&apos;t work, you should run that command in a
            terminal and then press the "Try Again" button:
          </p>
          <button class="button is-primary" on:click={copyAuthTokenFile}
            >Copy file</button
          >
        {:else}
          <p>
            Once you've copied the file, press the "Try Again" button to reload
            the file:
          </p>
        {/if}
        <button class="button is-primary" on:click={invalidateCaches}
          >Try Again</button
        >
      </div>
    {:else if authTokenStatus === 'success'}
      {#if nodeInfoStatus !== 'success'}
        <div class="content" in:slide|local>
          <p>
            In order for this program to work, your computer needs to talk to
            the ZeroTier service on port {SERVICE_PORT_ZT}, which should be
            secured by the auth token file in <code>{configPath}</code>. This
            program was able to open that file, but port {SERVICE_PORT_ZT} didn&apos;t
            respond as expected with the ZeroTier auth token. There are two likely
            reasons for this:
          </p>
          <ol>
            <li>
              The copy of <code>authtoken.secret</code> at
              <code>{configPath}</code>
              is different from the copy the ZeroTier service expects.
            </li>
            <li>
              The ZeroTier service is not running on port {SERVICE_PORT_ZT}, and
              instead some other program is running on it.
            </li>
          </ol>
          <p>Once you resolve this problem, press the "Try Again" button:</p>
          <button class="button is-primary" on:click={invalidateCaches}
            >Try Again</button
          >
        </div>
      {:else}
        <p in:slide|local>Loaded ZeroTier auth token.</p>
      {/if}
    {/if}
  </div>
</div>

<style>
</style>
