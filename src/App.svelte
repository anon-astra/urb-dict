<script>
  let query = $state("");
  let term = $state("");
  let items = $state([]);
  let status = $state("idle");
  let error = $state("");

  function linkify(text) {
    return (text ?? "")
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(
        /\[([^\]]+)\]/g,
        (_, word) =>
          `<a href="https://www.urbandictionary.com/define.php?term=${encodeURIComponent(word)}" target="_blank" rel="noreferrer">${word}</a>`
      )
      .replace(/\n/g, "<br />");
  }

  async function lookup(event) {
    event?.preventDefault();
    const q = query.trim();
    if (!q) return;
    status = "loading";
    error = "";
    term = q;
    try {
      const res = await fetch(
        `https://api.urbandictionary.com/v0/define?term=${encodeURIComponent(q)}`
      );
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      items = (data.list ?? [])
        .slice()
        .sort((a, b) => b.thumbs_up - b.thumbs_down - (a.thumbs_up - a.thumbs_down))
        .slice(0, 3);
      status = "done";
    } catch (err) {
      items = [];
      error = err.message || "Lookup failed";
      status = "error";
    }
  }
</script>

<main>
  <header>
    <p class="mark">UD</p>
    <h1>Three meanings.<br />Nothing else.</h1>
  </header>

  <form onsubmit={lookup}>
    <input
      bind:value={query}
      placeholder="word or phrase"
      autocomplete="off"
      spellcheck="false"
      aria-label="Term"
    />
    <button type="submit" disabled={status === "loading"}>
      {status === "loading" ? "…" : "Look up"}
    </button>
  </form>

  {#if status === "error"}
    <p class="note">{error}</p>
  {:else if status === "done" && items.length === 0}
    <p class="note">No definitions for “{term}”.</p>
  {:else if items.length}
    <section>
      <p class="term">{items[0].word}</p>
      <ol>
        {#each items as item, i (item.defid)}
          <li>
            <span class="n">{String(i + 1).padStart(2, "0")}</span>
            <div>
              <p class="def">{@html linkify(item.definition)}</p>
              {#if item.example}
                <p class="ex">{@html linkify(item.example)}</p>
              {/if}
              <p class="meta">
                <span>+{item.thumbs_up}</span>
                <span>−{item.thumbs_down}</span>
                <span>{item.author}</span>
              </p>
            </div>
          </li>
        {/each}
      </ol>
    </section>
  {/if}
</main>

<style>
  :global(*) {
    box-sizing: border-box;
  }
  :global(html, body) {
    margin: 0;
    background: #000;
    color: #f4f1ec;
    font-family: "Instrument Sans", sans-serif;
  }
  :global(body) {
    min-height: 100vh;
  }
  :global(#app) {
    min-height: 100vh;
  }

  main {
    max-width: 640px;
    margin: 0 auto;
    padding: 72px 24px 96px;
  }

  .mark {
    margin: 0 0 28px;
    color: #ff5a00;
    font-family: "IBM Plex Mono", monospace;
    font-size: 12px;
    letter-spacing: 0.22em;
  }

  h1 {
    margin: 0 0 36px;
    font-size: clamp(32px, 6vw, 48px);
    font-weight: 500;
    letter-spacing: -0.04em;
    line-height: 0.95;
  }

  form {
    display: flex;
    gap: 8px;
    border-bottom: 1px solid #ff5a00;
    padding-bottom: 10px;
  }

  input {
    flex: 1;
    background: transparent;
    border: 0;
    outline: none;
    color: #f4f1ec;
    font: 500 18px "Instrument Sans", sans-serif;
  }
  input::placeholder {
    color: #5c564e;
  }

  button {
    background: #ff5a00;
    color: #000;
    border: 0;
    padding: 8px 14px;
    font: 500 13px "IBM Plex Mono", monospace;
    letter-spacing: 0.04em;
    cursor: pointer;
  }
  button:disabled {
    opacity: 0.5;
    cursor: wait;
  }

  .note {
    margin-top: 28px;
    color: #ff5a00;
    font-family: "IBM Plex Mono", monospace;
    font-size: 13px;
  }

  .term {
    margin: 40px 0 8px;
    color: #ff5a00;
    font-size: 22px;
    font-weight: 600;
    letter-spacing: -0.03em;
  }

  ol {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  li {
    display: grid;
    grid-template-columns: 36px 1fr;
    gap: 12px;
    padding: 22px 0;
    border-top: 1px solid #1c1c1c;
  }

  .n {
    color: #ff5a00;
    font-family: "IBM Plex Mono", monospace;
    font-size: 12px;
    padding-top: 4px;
  }

  .def {
    margin: 0;
    font-size: 17px;
    line-height: 1.45;
  }
  .def :global(a),
  .ex :global(a) {
    color: #ff5a00;
    text-decoration: none;
  }

  .ex {
    margin: 10px 0 0;
    color: #9a9288;
    font-size: 14px;
    line-height: 1.45;
  }

  .meta {
    margin: 12px 0 0;
    display: flex;
    gap: 14px;
    color: #6b645c;
    font-family: "IBM Plex Mono", monospace;
    font-size: 11px;
  }
</style>
