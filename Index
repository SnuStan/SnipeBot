<!DOCTYPE html>
<html>
<head>
  <title>Solana Meme Sniper</title>
  <script src="https://unpkg.com/@solana/web3.js@1.73.2/lib/index.iife.min.js"></script>
  <style>
    body { font-family: Arial; padding: 20px; background: #111; color: #fff; }
    input, button { padding: 10px; margin-top: 10px; width: 100%; font-size: 16px; }
    button { background-color: #00ffae; border: none; color: #000; font-weight: bold; cursor: pointer; margin-bottom: 10px; }
    .token-button { background-color: #222; border: 1px solid #00ffae; color: #fff; }
  </style>
</head>
<body>
  <h2>Solana Meme Sniper</h2>

  <p>Token Mint Address:</p>
  <input type="text" id="tokenMint" placeholder="Paste token address">
  <p>Amount in SOL:</p>
  <input type="number" id="amount" placeholder="e.g. 0.1" value="0.1">
  <button onclick="snipe()">Snipe</button>

  <h3>New Meme Tokens (liq > 100 SOL):</h3>
  <div id="tokenList">Loading...</div>

  <script>
    const solMint = "So11111111111111111111111111111111111111112";

    async function snipe() {
      const provider = window.solana;
      if (!provider || !provider.isPhantom) {
        alert("Phantom wallet not found.");
        return;
      }

      const tokenMint = document.getElementById("tokenMint").value.trim();
      const amount = parseFloat(document.getElementById("amount").value || "0");

      if (!tokenMint || amount <= 0) {
        alert("Invalid token or amount.");
        return;
      }

      await provider.connect();
      const wallet = provider.publicKey;

      const quoteUrl = `https://quote-api.jup.ag/v6/quote?inputMint=${solMint}&outputMint=${tokenMint}&amount=${Math.floor(amount * 1e9)}&slippageBps=100`;
      const quoteRes = await fetch(quoteUrl);
      const quoteData = await quoteRes.json();
      if (!quoteData.data || quoteData.data.length === 0) {
        alert("No route found.");
        return;
      }

      const route = quoteData.data[0];

      const txRes = await fetch("https://quote-api.jup.ag/v6/swap", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          route,
          userPublicKey: wallet.toBase58(),
          wrapUnwrapSOL: true,
          feeAccount: null
        }),
      });
      const txData = await txRes.json();
      const transaction = solanaWeb3.Transaction.from(
        Uint8Array.from(txData.swapTransaction.data)
      );

      let signed = await provider.signTransaction(transaction);
      const connection = new solanaWeb3.Connection("https://api.mainnet-beta.solana.com");
      const txid = await connection.sendRawTransaction(signed.serialize());
      alert("Transaction sent: " + txid);
    }

    async function loadNewTokens() {
      const res = await fetch("https://public-api.birdeye.so/public/token/created?limit=20&sort=desc");
      const json = await res.json();
      const tokens = json.data;

      const container = document.getElementById("tokenList");
      container.innerHTML = "";

      for (let token of tokens) {
        if (token.liquidity >= 100) {
          const btn = document.createElement("button");
          btn.className = "token-button";
          btn.innerText = `${token.symbol || "?"} | ${token.liquidity.toFixed(1)} SOL`;
          btn.onclick = () => {
            document.getElementById("tokenMint").value = token.address;
            document.getElementById("amount").value = 0.1;
            snipe();
          };
          container.appendChild(btn);
        }
      }
    }

    loadNewTokens();
  </script>
</body>
</html>
