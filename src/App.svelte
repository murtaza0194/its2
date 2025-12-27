<script>
  import { fade, fly, slide } from "svelte/transition";
  import { quintOut } from "svelte/easing";

  let authCode = "";
  let token = "";
  let isLoggedIn = false;
  let qrCode = "";
  let hours = 0;
  $: cost = hours * 1000;

  function authenticate() {
    // @ts-ignore
    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        authCode = res.authCode;

        fetch("https://its.mouamle.space/api/auth-with-superQi", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            token: authCode,
          }),
        })
          .then((res) => res.json())
          .then((data) => {
            token = data.token;
            isLoggedIn = true;
            // @ts-ignore
            my.alert({
              content: "Login successful",
            });
          })
          .catch((err) => {
            let errorDetails = "";
            if (err && typeof err === "object") {
              errorDetails = JSON.stringify(err, null, 2);
            } else {
              errorDetails = String(err);
            }
            // @ts-ignore
            my.alert({
              content: "Error: " + errorDetails,
            });
          });
      },
      fail: (res) => {
        console.log(res.authErrorScopes);
      },
    });
  }

  function scan() {
    // @ts-ignore
    my.scan({
      type: "qr",
      success: (res) => {
        qrCode = res.code;
        // @ts-ignore
        my.alert({ title: "Spot Found" });
      },
    });
  }

  function pay() {
    if (!token) {
      // @ts-ignore
      my.alert({ content: "Please login first" });
      return;
    }

    fetch("https://its.mouamle.space/api/payment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: token,
      },
      body: JSON.stringify({
        hours: hours,
        amount: cost,
        parkingSpot: qrCode,
      }),
    })
      .then((res) => res.json())
      .then((data) => {
        // @ts-ignore
        my.tradePay({
          paymentUrl: data.url,
          success: (res) => {
            // @ts-ignore
            my.alert({
              content: "Payment successful",
            });
          },
        });
      })
      .catch((err) => {
        // @ts-ignore
        my.alert({
          content: "Payment failed",
        });
      });
  }
</script>

<div
  class="min-h-screen bg-slate-950 text-white flex flex-col relative overflow-hidden font-sans selection:bg-cyan-500/30"
>
  <!-- Cyberpunk Background Orbs -->
  <div
    class="absolute top-[10%] -left-10 w-72 h-72 bg-purple-600/20 rounded-full blur-[100px] pointer-events-none mix-blend-screen animate-pulse"
  ></div>
  <div
    class="absolute bottom-[10%] -right-10 w-80 h-80 bg-cyan-600/20 rounded-full blur-[100px] pointer-events-none mix-blend-screen animate-pulse delay-700"
  ></div>

  <!-- Glass Header -->
  <header
    class="w-full h-16 border-b border-white/5 bg-slate-950/50 backdrop-blur-xl sticky top-0 z-50 px-6 flex items-center justify-between"
  >
    <div class="flex items-center gap-3">
      <div class="relative">
        <div class="absolute inset-0 bg-cyan-500 blur-md opacity-50"></div>
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke-width="2"
          stroke="currentColor"
          class="w-6 h-6 text-cyan-400 relative z-10"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M21 7.5l-9-5.25L3 7.5m18 0l-9 5.25m9-5.25v9l-9 5.25M3 7.5l9 5.25M3 7.5v9l9 5.25m0-9v9"
          />
        </svg>
      </div>
      <span
        class="text-lg font-bold tracking-tight bg-gradient-to-r from-white to-slate-400 bg-clip-text text-transparent"
        >NeoPark</span
      >
    </div>
    {#if isLoggedIn}
      <div
        class="flex items-center gap-2 bg-emerald-500/10 border border-emerald-500/20 px-3 py-1 rounded-full"
        in:fade
      >
        <span class="relative flex h-2 w-2">
          <span
            class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"
          ></span>
          <span class="relative inline-flex rounded-full h-2 w-2 bg-emerald-500"
          ></span>
        </span>
        <span class="text-xs font-semibold text-emerald-400">Connected</span>
      </div>
    {/if}
  </header>

  <main
    class="flex-1 w-full max-w-lg mx-auto p-6 flex flex-col gap-8 relative z-10 justify-center"
  >
    <!-- LOGIN CARD -->
    {#if !isLoggedIn}
      <div
        class="flex flex-col items-center gap-10 text-center"
        in:fly={{ y: 30, duration: 800, easing: quintOut }}
      >
        <div class="relative flex items-center justify-center">
          <div
            class="absolute inset-0 border border-cyan-500/30 w-32 h-32 rounded-full animate-[spin_10s_linear_infinite]"
          ></div>
          <div
            class="absolute inset-0 border border-purple-500/30 w-32 h-32 rounded-full animate-[spin_15s_linear_infinite_reverse] scale-75"
          ></div>
          <div
            class="w-20 h-20 bg-gradient-to-br from-cyan-500 to-purple-600 rounded-full flex items-center justify-center shadow-[0_0_30px_rgba(34,211,238,0.4)]"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="2"
              stroke="currentColor"
              class="w-8 h-8 text-white"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M15.75 6a3.75 3.75 0 1 1-7.5 0 3.75 3.75 0 0 1 7.5 0ZM4.501 20.118a7.5 7.5 0 0 1 14.998 0A17.933 17.933 0 0 1 12 21.75c-2.676 0-5.216-.584-7.499-1.632Z"
              />
            </svg>
          </div>
        </div>

        <div class="space-y-2">
          <h1 class="text-4xl font-bold text-white tracking-tight">
            Access Granted
          </h1>
          <p class="text-slate-400 text-sm">
            Auth with SuperQi ID to initiate session.
          </p>
        </div>

        <button
          on:click={authenticate}
          class="group relative w-full bg-white text-slate-950 font-bold py-4 px-6 rounded-xl overflow-hidden transition-all hover:scale-[1.02] active:scale-[0.98]"
        >
          <div
            class="absolute inset-0 bg-gradient-to-r from-cyan-400 via-purple-400 to-cyan-400 opacity-0 group-hover:opacity-100 transition-opacity duration-500 bg-[length:200%_auto] animate-gradient"
          ></div>
          <span class="relative flex items-center justify-center gap-2">
            Initialize Login
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="2"
              stroke="currentColor"
              class="w-4 h-4"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M13.5 4.5 21 12m0 0-7.5 7.5M21 12H3"
              />
            </svg>
          </span>
        </button>
      </div>
    {/if}

    <!-- SCAN MODE -->
    {#if isLoggedIn && !qrCode}
      <div
        class="w-full bg-slate-900/60 backdrop-blur-2xl border border-white/10 rounded-3xl p-8 flex flex-col items-center gap-8 shadow-2xl"
        in:fly={{ y: 20, duration: 600 }}
      >
        <div class="flex flex-col items-center gap-4">
          <div
            class="relative w-full max-w-[200px] aspect-square rounded-2xl border-2 border-dashed border-slate-700 flex items-center justify-center group cursor-pointer hover:border-cyan-500/50 transition-colors"
            on:click={scan}
          >
            <!-- Scanner Line Animation -->
            <div
              class="absolute top-0 w-full h-1 bg-gradient-to-r from-transparent via-cyan-500 to-transparent shadow-[0_0_20px_#06b6d4] animate-[scan_2s_ease-in-out_infinite]"
            ></div>

            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-12 h-12 text-slate-500 group-hover:text-cyan-400 transition-colors"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M3.75 4.875c0-.621.504-1.125 1.125-1.125h4.5c.621 0 1.125.504 1.125 1.125v4.5c0 .621-.504 1.125-1.125 1.125h-4.5A1.125 1.125 0 0 1 3.75 9.375v-4.5ZM3.75 14.625c0-.621.504-1.125 1.125-1.125h4.5c.621 0 1.125.504 1.125 1.125v4.5c0 .621-.504 1.125-1.125 1.125h-4.5a1.125 1.125 0 0 1-1.125-1.125v-4.5ZM13.5 4.875c0-.621.504-1.125 1.125-1.125h4.5c.621 0 1.125.504 1.125 1.125v4.5c0 .621-.504 1.125-1.125 1.125h-4.5A1.125 1.125 0 0 1 13.5 9.375v-4.5Z"
              />
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M6.75 6.75h.75v.75h-.75v-.75ZM6.75 16.5h.75v.75h-.75v-.75ZM16.5 6.75h.75v.75h-.75v-.75ZM13.5 13.5h.75v.75h-.75v-.75ZM13.5 19.5h.75v.75h-.75v-.75ZM19.5 13.5h.75v.75h-.75v-.75ZM19.5 19.5h.75v.75h-.75v-.75ZM16.5 16.5h.75v.75h-.75v-.75Z"
              />
            </svg>
          </div>
          <div class="text-center">
            <h2 class="text-xl font-bold text-white">Detect Parking Spot</h2>
            <p class="text-slate-400 text-sm mt-1">Point camera at QR Code</p>
          </div>
        </div>

        <button
          on:click={scan}
          class="w-full bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-3.5 rounded-xl transition-all shadow-[0_0_20px_rgba(8,145,178,0.4)]"
        >
          Activate Scanner
        </button>
      </div>
    {/if}

    <!-- PAYMENT INTERFACE -->
    {#if isLoggedIn && qrCode}
      <div class="flex flex-col gap-5" in:slide={{ duration: 500, axis: "y" }}>
        <!-- Spot Tag -->
        <div
          class="bg-gradient-to-r from-slate-900 to-slate-800 border border-white/10 p-5 rounded-2xl flex items-center justify-between"
        >
          <div class="flex flex-col">
            <span
              class="text-[10px] uppercase font-bold tracking-widest text-cyan-400 mb-1"
              >Target Spot</span
            >
            <span class="font-mono text-2xl font-bold tracking-wider"
              >{qrCode}</span
            >
          </div>
          <button
            class="bg-white/5 hover:bg-white/10 p-2 rounded-lg transition-colors border border-white/5"
            on:click={() => (qrCode = "")}
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-5 h-5 text-slate-300"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99"
              />
            </svg>
          </button>
        </div>

        <!-- Cost Calculator -->
        <div
          class="bg-slate-900/80 backdrop-blur-md border border-white/10 p-6 rounded-3xl flex flex-col gap-6 shadow-2xl"
        >
          <div class="flex items-center justify-between">
            <label class="text-sm font-semibold text-slate-300"
              >Parking Duration</label
            >
            <span class="text-xs bg-white/5 px-2 py-1 rounded text-slate-400"
              >Hours</span
            >
          </div>

          <div class="relative group">
            <div
              class="absolute -inset-0.5 bg-gradient-to-r from-cyan-500 to-purple-500 rounded-xl opacity-20 group-focus-within:opacity-100 transition-opacity duration-300 blur"
            ></div>
            <input
              type="number"
              min="0"
              bind:value={hours}
              class="relative w-full bg-slate-950 text-center text-4xl font-bold py-6 rounded-xl outline-none text-white placeholder-slate-700"
              placeholder="0"
            />
          </div>

          <div class="h-px bg-white/10 w-full"></div>

          <div class="flex items-end justify-between">
            <span class="text-slate-400 text-sm font-medium"
              >Estimated<br />Total</span
            >
            <div class="text-right">
              <div
                class="text-3xl font-bold text-transparent bg-clip-text bg-gradient-to-l from-white to-slate-400"
              >
                {cost.toLocaleString()}
                <span class="text-lg text-slate-500">IQD</span>
              </div>
              <div class="text-[10px] text-emerald-400 font-mono mt-1">
                RATE: 1,000 IQD/HR
              </div>
            </div>
          </div>

          <button
            on:click={pay}
            disabled={hours <= 0}
            class="w-full mt-2 py-4 rounded-xl bg-gradient-to-r from-cyan-600 to-blue-600 text-white font-bold shadow-lg shadow-cyan-900/40 hover:shadow-cyan-500/20 hover:scale-[1.01] active:scale-[0.98] transition-all disabled:opacity-50 disabled:cursor-not-allowed disabled:shadow-none relative overflow-hidden group"
          >
            <div
              class="absolute inset-0 bg-white/20 translate-y-full group-hover:translate-y-0 transition-transform duration-300"
            ></div>
            <span class="relative">Confirm & Pay</span>
          </button>
        </div>
      </div>
    {/if}
  </main>

  {#if authCode}
    <div class="py-4 text-center">
      <code class="text-[10px] text-slate-600 font-mono tracking-widest"
        >{authCode.slice(0, 8)}••••••••</code
      >
    </div>
  {/if}
</div>

<style>
  :global(body) {
    background-color: #020617;
  }
</style>
