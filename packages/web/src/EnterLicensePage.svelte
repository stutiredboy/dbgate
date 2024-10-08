<script lang="ts">
  import { onMount } from 'svelte';
  import { useConfig } from './utility/metadataLoaders';
  import ErrorInfo from './elements/ErrorInfo.svelte';
  import Link from './elements/Link.svelte';
  import { internalRedirectTo } from './clientAuth';
  import TextAreaField from './forms/TextAreaField.svelte';
  import { writable } from 'svelte/store';
  import FormProviderCore from './forms/FormProviderCore.svelte';
  import FormTextAreaField from './forms/FormTextAreaField.svelte';
  import FormSubmit from './forms/FormSubmit.svelte';
  import { apiCall } from './utility/api';
  import FormStyledButton from './buttons/FormStyledButton.svelte';
  import getElectron from './utility/getElectron';

  const config = useConfig();
  const values = writable({ amoid: null, databaseServer: null });

  let errorMessage = '';

  onMount(() => {
    const removed = document.getElementById('starting_dbgate_zero');
    if (removed) removed.remove();
  });
</script>

<FormProviderCore {values}>
  <div class="root theme-light theme-type-light">
    <div class="text">DbGate</div>
    <div class="wrap">
      <div class="logo">
        <img class="img" src="logo192.png" />
      </div>
      <div class="box">
        <div class="heading">License</div>
        <FormTextAreaField label="Enter your license key" name="licenseKey" rows={5} />

        <div class="submit">
          <FormSubmit
            value="Save license"
            on:click={async e => {
              const { licenseKey } = e.detail;
              const resp = await apiCall('config/save-license-key', { licenseKey });
              if (resp?.status == 'ok') {
                internalRedirectTo('/index.html');
              } else {
                errorMessage = resp?.errorMessage || 'Error saving license key';
              }
            }}
          />
        </div>

        <div class="submit">
          <FormStyledButton
            value="Start 30-day trial"
            on:click={async e => {
              errorMessage = '';
              const license = await apiCall('config/start-trial');
              if (license?.status == 'ok') {
                internalRedirectTo('/index.html');
              } else {
                errorMessage = license?.errorMessage || 'Error starting trial';
              }
            }}
          />
        </div>

        {#if getElectron()}
          <div class="submit">
            <FormStyledButton
              value="Exit"
              on:click={e => {
                getElectron().send('quit-app');
              }}
            />
          </div>
        {/if}

        {#if errorMessage}
          <div class="error">{errorMessage}</div>
        {/if}

        <div class="purchase-info">
          If you want to purchase Premium license, please contact us at <Link href="mailto:sales@dbgate.eu"
            >sales@dbgate.eu</Link
          >
        </div>
      </div>
    </div>
  </div>
</FormProviderCore>

<style>
  .logo {
    display: flex;
    margin-bottom: 1rem;
    align-items: center;
    justify-content: center;
  }
  .img {
    width: 80px;
  }
  .text {
    position: fixed;
    top: 1rem;
    left: 1rem;
    font-size: 30pt;
    font-family: monospace;
    color: var(--theme-bg-2);
    text-transform: uppercase;
  }

  .root {
    color: var(--theme-font-1);
    display: flex;
    justify-content: center;
    background-color: var(--theme-bg-1);
    align-items: baseline;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }

  .box {
    width: 600px;
    max-width: 80vw;
    /* max-width: 600px;
        width: 40vw; */
    border: 1px solid var(--theme-border);
    border-radius: 4px;
    background-color: var(--theme-bg-0);
  }

  .wrap {
    margin-top: 20vh;
  }

  .heading {
    text-align: center;
    margin: 1em;
    font-size: xx-large;
  }

  .submit {
    margin: var(--dim-large-form-margin);
    display: flex;
  }

  .submit :global(input) {
    flex: 1;
    font-size: larger;
  }

  .error {
    margin: var(--dim-large-form-margin);
    color: red;
  }

  .purchase-info {
    margin: var(--dim-large-form-margin);
  }
</style>
