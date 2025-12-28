<script>
  import { fade, fly, slide, scale } from "svelte/transition";
  import { quintOut, elasticOut } from "svelte/easing";

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

  function handleKeydown(event, action) {
    if (event.key === "Enter" || event.key === " ") {
      action();
    }
  }
</script>

<div
  class="min-h-screen flex flex-col relative font-sans text-slate-900 bg-slate-50 selection:bg-indigo-500/20 overflow-hidden"
>
  <!-- Premium Mesh Gradient Background -->
  <div class="fixed inset-0 z-0 pointer-events-none">
    <div
      class="absolute top-[-10%] left-[-10%] w-[500px] h-[500px] bg-indigo-200/40 rounded-full blur-[120px] mix-blend-multiply animate-blob"
    ></div>
    <div
      class="absolute top-[10%] right-[-10%] w-[500px] h-[500px] bg-purple-200/40 rounded-full blur-[120px] mix-blend-multiply animate-blob animation-delay-2000"
    ></div>
    <div
      class="absolute bottom-[-10%] left-[20%] w-[500px] h-[500px] bg-blue-200/40 rounded-full blur-[120px] mix-blend-multiply animate-blob animation-delay-4000"
    ></div>
  </div>

  <!-- Clean Header -->
  <header
    class="w-full h-18 bg-white/80 backdrop-blur-xl sticky top-0 z-50 px-6 flex items-center justify-between border-b border-white/60 shadow-sm"
  >
    <div class="flex items-center gap-3">
      <div
        class="w-10 h-10 bg-gradient-to-br from-indigo-600 to-violet-600 rounded-xl flex items-center justify-center text-white shadow-lg shadow-indigo-200 rotate-3"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke-width="2.5"
          stroke="currentColor"
          class="w-5 h-5 -rotate-3"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M21 7.5l-9-5.25L3 7.5m18 0l-9 5.25m9-5.25v9l-9 5.25M3 7.5l9 5.25M3 7.5v9l9 5.25m0-9v9"
          />
        </svg>
      </div>
      <span class="text-xl font-extrabold tracking-tight text-slate-800"
        >NeoPark</span
      >
    </div>
    {#if isLoggedIn}
      <div
        class="flex items-center gap-2 bg-emerald-50/80 border border-emerald-100 px-3 py-1.5 rounded-full backdrop-blur-md"
        in:scale={{ duration: 300, start: 0.9 }}
      >
        <span class="relative flex h-2.5 w-2.5">
          <span
            class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"
          ></span>
          <span
            class="relative inline-flex rounded-full h-2.5 w-2.5 bg-emerald-500"
          ></span>
        </span>
        <span class="text-xs font-bold text-emerald-700 tracking-wide"
          >ONLINE</span
        >
      </div>
    {/if}
  </header>

  <main
    class="flex-1 w-full max-w-lg mx-auto p-6 flex flex-col gap-6 relative z-10 justify-center"
  >
    <!-- LOGIN STATE -->
    {#if !isLoggedIn}
      <div
        class="flex flex-col items-center gap-8 text-center py-6"
        in:fly={{ y: 20, duration: 600, easing: quintOut }}
      >
        <div class="relative">
          <div
            class="absolute inset-0 bg-indigo-500/20 blur-3xl rounded-full"
          ></div>
          <div
            class="w-28 h-28 bg-white/90 backdrop-blur-2xl rounded-[2rem] flex items-center justify-center shadow-[0_20px_50px_-12px_rgba(0,0,0,0.1)] border border-white/60 relative z-10"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-14 h-14 text-indigo-600"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M15.75 6a3.75 3.75 0 1 1-7.5 0 3.75 3.75 0 0 1 7.5 0ZM4.501 20.118a7.5 7.5 0 0 1 14.998 0A17.933 17.933 0 0 1 12 21.75c-2.676 0-5.216-.584-7.499-1.632Z"
              />
            </svg>
          </div>
        </div>

        <div class="space-y-4 max-w-xs mx-auto">
          <h1 class="text-4xl font-extrabold text-slate-900 tracking-tight">
            Welcome Back
          </h1>
          <p class="text-slate-500 text-base font-medium leading-relaxed">
            Sign in with your SuperQi ID to manage your parking sessions
            effortlessly.
          </p>
        </div>

        <button
          on:click={authenticate}
          class="w-full bg-gradient-to-r from-indigo-600 to-violet-600 text-white font-bold text-lg py-4 px-6 rounded-2xl shadow-[0_10px_30px_-10px_rgba(79,70,229,0.5)] hover:shadow-[0_20px_40px_-10px_rgba(79,70,229,0.6)] hover:scale-[1.02] active:scale-[0.98] transition-all flex items-center justify-center gap-3 mt-4 border border-indigo-500/20"
        >
          <span>Connect SuperQi</span>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="2.5"
            stroke="currentColor"
            class="w-5 h-5"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M13.5 4.5 21 12m0 0-7.5 7.5M21 12H3"
            />
          </svg>
        </button>
      </div>
    {/if}

    <!-- SCANNER STATE -->
    {#if isLoggedIn && !qrCode}
      <div
        class="bg-white/80 backdrop-blur-xl rounded-[2.5rem] p-8 flex flex-col items-center gap-8 shadow-[0_20px_60px_-15px_rgba(0,0,0,0.05)] border border-white/60"
        in:fly={{ y: 20, duration: 500 }}
      >
        <div class="text-center space-y-2">
          <h2 class="text-2xl font-bold text-slate-900">Start Parking</h2>
          <p class="text-slate-500 font-medium">
            Scan the QR code at your location
          </p>
        </div>

        <div
          class="relative w-72 h-72 bg-slate-50/50 rounded-3xl border-3 border-dashed border-slate-200 flex flex-col items-center justify-center cursor-pointer hover:border-indigo-400 hover:bg-indigo-50/30 transition-all group gap-4"
          role="button"
          tabindex="0"
          on:click={scan}
          on:keydown={(e) => handleKeydown(e, scan)}
        >
          <div
            class="w-20 h-20 bg-white rounded-2xl shadow-lg shadow-slate-100 flex items-center justify-center group-hover:scale-110 transition-transform duration-300"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-10 h-10 text-indigo-600"
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
          <span
            class="text-base font-semibold text-slate-500 group-hover:text-indigo-600"
            >Tap to Scan</span
          >
        </div>

        <button
          on:click={scan}
          class="w-full bg-gradient-to-r from-indigo-600 to-violet-600 text-white font-bold text-lg py-4 px-6 rounded-2xl shadow-[0_10px_30px_-10px_rgba(79,70,229,0.5)] hover:shadow-[0_20px_40px_-10px_rgba(79,70,229,0.6)] hover:scale-[1.02] active:scale-[0.98] transition-all flex items-center justify-center gap-3 border border-indigo-500/20"
        >
          Open Camera
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="2"
            stroke="currentColor"
            class="w-5 h-5"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M6.827 6.175A2.31 2.31 0 0 1 5.186 7.23c-.38.054-.757.112-1.134.175C2.999 7.58 2.25 8.507 2.25 9.574V18a2.25 2.25 0 0 0 2.25 2.25h15A2.25 2.25 0 0 0 21.75 18V9.574c0-1.067-.75-1.994-1.802-2.169a47.865 47.865 0 0 0-1.134-.175 2.31 2.31 0 0 1-1.64-1.055l-.822-1.316a2.192 2.192 0 0 0-1.736-1.039 48.774 48.774 0 0 0-5.232 0 2.192 2.192 0 0 0-1.736 1.039l-.821 1.316Z"
            />
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M16.5 12.75a4.5 4.5 0 1 1-9 0 4.5 4.5 0 0 1 9 0ZM18.75 10.5h.008v.008h-.008V10.5Z"
            />
          </svg>
        </button>
      </div>
    {/if}

    <!-- PAYMENT DETAILS -->
    {#if isLoggedIn && qrCode}
      <div class="flex flex-col gap-6" in:slide={{ duration: 400, axis: "y" }}>
        <!-- Spot Info -->
        <div
          class="bg-white/60 backdrop-blur-md border border-white/50 p-5 rounded-2xl flex items-center justify-between shadow-sm"
        >
          <div class="flex items-center gap-4">
            <div
              class="w-12 h-12 bg-orange-100 rounded-xl flex items-center justify-center text-orange-600 shadow-sm"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke-width="2"
                stroke="currentColor"
                class="w-6 h-6"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M15 10.5a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z"
                />
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M19.5 10.5c0 7.142-7.5 11.25-7.5 11.25S4.5 17.642 4.5 10.5a7.5 7.5 0 1 1 15 0Z"
                />
              </svg>
            </div>
            <div>
              <p
                class="text-xs text-slate-500 font-bold uppercase tracking-wider"
              >
                Spot ID
              </p>
              <p
                class="text-xl font-extrabold text-slate-900 font-mono tracking-tight"
              >
                {qrCode}
              </p>
            </div>
          </div>
          <button
            class="text-sm font-bold text-indigo-600 hover:text-indigo-700 px-4 py-2 bg-indigo-50 hover:bg-indigo-100 rounded-lg transition-colors"
            on:click={() => (qrCode = "")}
          >
            Change
          </button>
        </div>

        <!-- Calculator -->
        <div
          class="bg-white/90 backdrop-blur-2xl p-7 rounded-[2.5rem] shadow-[0_20px_60px_-15px_rgba(0,0,0,0.05)] border border-white/60 flex flex-col gap-8"
        >
          <div class="flex flex-col gap-3">
            <label
              for="hours-input"
              class="text-sm font-bold text-slate-700 ml-1"
              >Parking Duration</label
            >
            <div class="relative group">
              <input
                id="hours-input"
                type="number"
                min="0"
                bind:value={hours}
                class="w-full bg-slate-50/50 border-2 border-slate-200 focus:border-indigo-500 px-6 py-5 rounded-2xl text-3xl font-bold text-slate-900 outline-none transition-all text-center placeholder-slate-300 focus:bg-white focus:shadow-[0_0_0_4px_rgba(99,102,241,0.1)]"
                placeholder="0"
              />
              <div
                class="absolute right-6 top-1/2 -translate-y-1/2 text-slate-400 font-bold pointer-events-none text-sm tracking-wider"
              >
                HOURS
              </div>
            </div>
          </div>

          <div
            class="bg-indigo-50/60 rounded-3xl p-6 border border-indigo-100/50"
          >
            <div class="flex justify-between items-end">
              <span class="text-slate-600 font-semibold mb-1">Total Amount</span
              >
              <div class="text-right">
                <span class="text-4xl font-extrabold text-slate-900"
                  >{cost.toLocaleString()}</span
                >
                <span class="text-xl font-bold text-slate-500 ml-1">IQD</span>
              </div>
            </div>
            <div
              class="mt-3 pt-3 border-t border-indigo-100 flex items-center justify-between"
            >
              <div class="text-xs font-semibold text-indigo-400">
                Rate Calculation
              </div>
              <div
                class="text-xs font-bold text-indigo-700 flex items-center gap-1.5 bg-white px-2 py-1 rounded-md shadow-sm"
              >
                <span class="w-2 h-2 rounded-full bg-indigo-500"></span>
                1,000 IQD / Hour
              </div>
            </div>
          </div>

          <button
            on:click={pay}
            disabled={hours <= 0}
            class="w-full py-5 rounded-2xl bg-gradient-to-r from-indigo-600 to-violet-600 text-white font-bold text-lg shadow-[0_10px_30px_-10px_rgba(79,70,229,0.5)] hover:shadow-[0_20px_40px_-10px_rgba(79,70,229,0.6)] hover:scale-[1.01] active:scale-[0.98] transition-all disabled:opacity-50 disabled:cursor-not-allowed disabled:shadow-none border border-indigo-500/20"
          >
            Confirm & Pay
          </button>
        </div>
      </div>
    {/if}
  </main>

  {#if authCode}
    <div class="py-6 text-center opacity-40">
      <code class="text-[10px] text-slate-400 font-mono tracking-widest"
        >{authCode.slice(0, 8)}••••••••</code
      >
    </div>
  {/if}
</div>

<style>
  @keyframes blob {
    0% {
      transform: translate(0px, 0px) scale(1);
    }
    33% {
      transform: translate(30px, -50px) scale(1.1);
    }
    66% {
      transform: translate(-20px, 20px) scale(0.9);
    }
    100% {
      transform: translate(0px, 0px) scale(1);
    }
  }
  .animate-blob {
    animation: blob 7s infinite;
  }
  .animation-delay-2000 {
    animation-delay: 2s;
  }
  .animation-delay-4000 {
    animation-delay: 4s;
  }
</style>
