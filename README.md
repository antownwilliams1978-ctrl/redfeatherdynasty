# Red Feather Dynasty — Sovereign Digital Domain

This repository contains a small static site for Red Feather Dynasty (landing page, lore/media pages, and audio assets). It is intended to be hosted as a static site (GitHub Pages or any static host) and includes a Web3 gating snippet that verifies ownership of a token before redirecting visitors to an IPFS vault.

## Quick summary
- Static HTML/CSS/JS (no build step).
- Pages: `index.html`, `lore.html`, `media.html`.
- Audio assets: `ACT_*.mp3` used by `lore.html`.
- Admin UI: `admin/index.html` loads the Sveltia client-side CMS (no backend included).
- Custom domain: `CNAME` present (redfeatherdynasty.com).

---

## Deploying to GitHub Pages
These steps will publish this repository as a website using GitHub Pages.

1. Push the repo to GitHub (already done).
2. Go to the repository on GitHub > Settings > Pages.
3. Under "Build and deployment" choose "GitHub Pages" and select the branch `main` and "/ (root)" as the folder (this repo is static, no build required).
4. If `CNAME` is present (it is), GitHub will attempt to use that custom domain. Verify DNS:
   - If your domain is managed on a DNS provider, create records recommended by GitHub Pages:
     - For an apex domain (example.com): create A records pointing to GitHub's Pages IPs (check current IPs in GitHub Pages docs).
     - For a subdomain (www.example.com): create a CNAME record pointing to `<username>.github.io` or the GitHub Pages target.
   - Wait for DNS propagation; in the Pages settings GitHub will report the domain status.
5. After Pages builds, visit the site URL. If HTTPS is available, enable it in the Pages settings.

Notes:
- Because the repo includes a `CNAME` file, GitHub Pages will try to serve the site from that domain; if you change the domain, update or remove the `CNAME` file.

## Running locally
To test the site locally, serve the static files from the repo root. Examples:

- Python 3 (quick test):

  ```bash
  python -m http.server 8000
  # open http://localhost:8000/index.html
  ```

- Using `serve` (node):

  ```bash
  npx serve .
  ```

No build step is required.

## Replacing the Web3 configuration (contractAddress & vaultUrl)
The Web3 gating logic is client-side and lives in `index.html` (search for `contractAddress` / `vaultUrl`). To change the values:

1. Open `index.html`.
2. Find the block near the bottom that starts with the comment `<!-- WEB3 GATING LOGIC -->` or locate these variables:

```js
const contractAddress = "0x9b5fb710b27ffd9f7c3e126fa76c2c7cbcf18a97";
const vaultUrl = "https://bafkreibi6v6wplsltgz5aqs5j3swu475hv7su3acma3mxm5h5einw2gfny.ipfs.inbrowser.link/";
```

3. Replace `contractAddress` with the target ERC-20/ERC-721/ERC-1155 contract address you want to gate on.
4. Replace `vaultUrl` with the destination URL you want verified holders to be redirected to (IPFS gateway, another page, or a hosted asset).

Security and behavior notes:
- These values are public (client-side). Do not put secrets or private keys here — the gate only *checks* on-chain ownership and is visible to anyone.
- The script uses the `balanceOf(address)` ABI signature; ensure your contract implements that method (ERC-20/ERC-721-compatible). For ERC-1155 or other token standards the check may need to change (e.g., `balanceOf` with `id`, or `ownerOf` for ERC-721 metadata). Update the `abi` variable accordingly.
- The script connects to Polygon via `https://polygon-rpc.com`. If you prefer a different provider (Infura, Alchemy, QuickNode), update the provider URL and/or use `window.ethereum` provider instead for network operations.

## Testing the Web3 gate locally
1. Open the site in a desktop browser with a Web3 wallet extension (MetaMask, Coinbase Wallet extension, etc.).
2. Click the "🔐 Enter Web3 Vault" button on the home page.
3. The page will prompt to connect the wallet and then attempt to read `balanceOf(userAddress)` on the configured contract. If the balance is > 0 it redirects to `vaultUrl`.

If you see "No Web3 Wallet detected" the browser doesn't expose `window.ethereum` — MetaMask or another extension is required for the direct connect flow.

## Updating Form endpoints and admin CMS
- The contact form in `index.html` posts to Formspree (`https://formspree.io/f/maqrrenn`). To use your own Formspree form or a different service, change the `action` attribute on the `<form>` element.
- `admin/index.html` simply loads the Sveltia CMS JS from unpkg. There is no server-side persistence configured in the repo — Sveltia likely expects further configuration (API keys or a serverless backend). If you want, I can inspect Sveltia docs and add an example configuration for storing content (IPFS, Netlify functions, or a simple JSON backend).

## Wiring video placeholders (media embeds)
index.html contains a small YAML-like front-matter header at the top with keys like `genesis_video` and `podcast_link`. The visible video placeholders in `index.html` and `media.html` are currently inert. To wire them automatically:

- Option A — Simple manual change: replace the `video-placeholder` DOM nodes with an `<iframe src="https://www.youtube.com/embed/<VIDEO_ID>" ...>` where you paste your YouTube embed URLs.
- Option B — Use small client-side JS to read the front-matter values at the top of `index.html` (or move them into a JSON `<script type="application/json" id="site-meta">` block) and dynamically inject iframes into the `.video-wrapper` elements.

Example dynamic injection (you would add this script to `index.html`):

```html
<script>
  // Example: replace placeholder with embed from a front-matter variable
  const genesisVideo = "https://youtu.be/196Ww6FFW6E"; // update as needed
  const podcastLink = "https://youtube.com/@twofeathers_chiefredfeather78";

  function toEmbed(url){
    try{
      const u = new URL(url);
      // Handle short youtu.be links
      if(u.hostname === 'youtu.be') return 'https://www.youtube.com/embed/' + u.pathname.slice(1);
      if(u.hostname.includes('youtube.com')){
        const p = new URLSearchParams(u.search);
        if(p.has('v')) return 'https://www.youtube.com/embed/' + p.get('v');
        // handle channel/playlist links — requires manual mapping
      }
      return url; // fallback
    }catch(e){ return url; }
  }

  document.addEventListener('DOMContentLoaded', ()=>{
    const genesisEmbed = toEmbed(genesisVideo);
    const podEmbed = toEmbed(podcastLink);
    const g = document.querySelector('#genesisPlay');
    const p = document.querySelector('#podcastPlay');
    if(g) g.innerHTML = `<iframe src="${genesisEmbed}" allowfullscreen></iframe>`;
    if(p) p.innerHTML = `<iframe src="${podEmbed}" allowfullscreen></iframe>`;
  });
</script>
```

Would you like me to implement automatic wiring of the two placeholders (Genesis + Podcast) by reading the front-matter values and injecting the correct YouTube embeds? If yes, tell me which pages you want updated (index.html only, media.html, or both) and whether to commit the changes directly.

## Support & next steps I can take
- Add a `README.md` (this file) — done.
- Replace the hard-coded contract address and vault URL with environment-driven values or a small JSON config file (I can add `site-config.json` and update `index.html` to read it).
- Wire the video placeholders automatically (implement and commit the JS changes).
- Add simple CI or GitHub Actions to validate that `CNAME` and `index.html` exist before deploying.

If you'd like me to proceed with wiring video embeds or updating `index.html` to read a `site-config.json`, tell me which option you prefer and I will implement it and commit the changes.