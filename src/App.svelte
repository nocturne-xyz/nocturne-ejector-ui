<script>
  import { ethers } from "ethers";

  let spendKey = "";
  let copied = false;

  // ! ⚠️⚠️⚠️ THIS MESSAGE CANNOT BE CHANGED! KEY DERIVATION IS DICTATED BY THE EXACT TEXT ⚠️⚠️⚠️
  const SPEND_KEY_FIXED_MESSAGE = `Sign to generate your Nocturne spending key. This key will secure your funds in Nocturne.

By signing this message, I assert that
1. I trust the application
2. I have safely stored the private key (or seed phrase from which the private key was derived) for the connected Ethereum account
3. The only way I can recover access to my Nocturne account is by signing this message again with the same Ethereum account mentioned in #2.`;

  async function generateNocturneSpendKeyFromEoaSig(signer) {
    console.log(ethers);
    return ethers.keccak256(
      await signer.signMessage(Buffer.from(SPEND_KEY_FIXED_MESSAGE))
    );
  }

  async function getSigner() {
    const { ethereum } = window;
    if (!ethereum) {
      alert("Please install MetaMask!");
      return;
    }
    const provider = new ethers.BrowserProvider(ethereum);
    return provider.getSigner();
  }

  async function copyToClipboard() {
    if (spendKey) {
      await navigator.clipboard.writeText(spendKey);
      copied = true;
    }
  }

  async function handleSpendKeyClick() {
    try {
      const signer = await getSigner();
      if (!signer) {
        return;
      }

      spendKey = await generateNocturneSpendKeyFromEoaSig(signer);
    } catch (e) {
      console.error(e);
      await navigator.clipboard.writeText(e.toString());
      alert(`Failed to generate spend key, error copied to clipboard. \n ${e}`);
    }
  }
</script>

<main>
  <h2>Export Nocturne Spend Private Key</h2>
  <p>To export your spend private key:</p>
  <ol>
      <li>In MetaMask, switch to the Ethereum account you used to derive your Nocturne account the first time you used the app. If you're unsure, see our <a href="https://nocturne-xyz.gitbook.io/nocturne/users/metamask-snap">docs</a>.</li>
      <li>Press the button below to sign the key generation message.</li>
      <li>Copy your spend private key.</li>
      <li>Paste it into your <code>.env</code> file in the CLI repo.</li>
  </ol>
  <button on:click={handleSpendKeyClick}> Re-Generate Spend Private Key </button>
  {#if spendKey}
    <p>Spend Private Key: {spendKey}</p>
    <button on:click={copyToClipboard}> Copy to Clipboard </button>
    {#if copied}
      <p>Copied to clipboard!</p>
    {/if}
  {/if}
</main>

<style>
  main {
    text-align: left;
    padding: 1em;
    max-width: 480px;
    margin: 0 auto;
  }
  button {
    margin: 10px;
    padding: 1em;
  }
</style>
