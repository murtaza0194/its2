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

        // Mocking interaction for visual feedback
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
  class="min-h-screen flex flex-col relative font-sans text-slate-900 bg-[#FAFAFA]"
>
  <!-- Clean Header -->
  <header
    class="w-full h-16 bg-white sticky top-0 z-50 px-6 flex items-center justify-between shadow-[0_2px_15px_-3px_rgba(0,0,0,0.07),0_10px_20px_-2px_rgba(0,0,0,0.04)]"
  >
    <div class="flex items-center gap-2.5">
      <div
        class="w-9 h-9 bg-indigo-600 rounded-xl flex items-center justify-center text-white shadow-md shadow-indigo-200"
      >
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
            d="M21 7.5l-9-5.25L3 7.5m18 0l-9 5.25m9-5.25v9l-9 5.25M3 7.5l9 5.25M3 7.5v9l9 5.25m0-9v9"
          />
        </svg>
      </div>
      <span class="text-lg font-bold tracking-tight text-slate-800"
        >NeoPark</span
      >
    </div>
    {#if isLoggedIn}
      <div
        class="flex items-center gap-1.5 bg-emerald-50 border border-emerald-100 px-3 py-1.5 rounded-full"
        in:scale={{ duration: 300, start: 0.9 }}
      >
        <span class="relative flex h-2 w-2">
          <span
            class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"
          ></span>
          <span class="relative inline-flex rounded-full h-2 w-2 bg-emerald-500"
          ></span>
        </span>
        <span class="text-xs font-semibold text-emerald-700">Online</span>
      </div>
    {/if}
  </header>

  <main
    class="flex-1 w-full max-w-lg mx-auto p-6 flex flex-col gap-8 relative z-10 justify-center"
  >
    <!-- LOGIN STATE -->
    {#if !isLoggedIn}
      <div
        class="flex flex-col items-center gap-8 text-center py-10"
        in:fly={{ y: 20, duration: 600, easing: quintOut }}
      >
        <div
          class="w-24 h-24 bg-white rounded-3xl flex items-center justify-center shadow-[0_8px_30px_rgb(0,0,0,0.04)] mb-4"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="w-12 h-12 text-indigo-600"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M15.75 6a3.75 3.75 0 1 1-7.5 0 3.75 3.75 0 0 1 7.5 0ZM4.501 20.118a7.5 7.5 0 0 1 14.998 0A17.933 17.933 0 0 1 12 21.75c-2.676 0-5.216-.584-7.499-1.632Z"
            />
          </svg>
        </div>

        <div class="space-y-3 max-w-xs mx-auto">
          <h1 class="text-3xl font-bold text-slate-900 tracking-tight">
            Welcome Back
          </h1>
          <p class="text-slate-500 leading-relaxed">
            Sign in with your SuperQi ID to manage your parking sessions
            effortlessly.
          </p>
        </div>

        <button
          on:click={authenticate}
          class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-4 px-6 rounded-xl shadow-lg shadow-indigo-200 transition-all hover:scale-[1.02] active:scale-[0.98] flex items-center justify-center gap-2 mt-4"
        >
          <span>Connect SuperQi</span>
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
        </button>
      </div>
    {/if}

    <!-- SCANNER STATE -->
    {#if isLoggedIn && !qrCode}
      <div
        class="bg-white rounded-3xl p-8 flex flex-col items-center gap-8 shadow-[0_8px_30px_rgb(0,0,0,0.04)] border border-slate-100"
        in:fly={{ y: 20, duration: 500 }}
      >
        <div class="text-center space-y-1">
          <h2 class="text-xl font-bold text-slate-900">Start Parking</h2>
          <p class="text-slate-500 text-sm">
            Scan the QR code at your location
          </p>
        </div>

        <div
          class="relative w-64 h-64 bg-slate-50 rounded-2xl border-2 border-dashed border-slate-200 flex flex-col items-center justify-center cursor-pointer hover:border-indigo-400 hover:bg-indigo-50/30 transition-all group gap-3"
          on:click={scan}
        >
          <div
            class="w-16 h-16 bg-white rounded-full shadow-sm flex items-center justify-center group-hover:scale-110 transition-transform duration-300"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-8 h-8 text-indigo-600"
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
            class="text-sm font-medium text-slate-500 group-hover:text-indigo-600"
            >Tap to Scan</span
          >
        </div>

        <button
          on:click={scan}
          class="w-full bg-slate-900 text-white font-semibold py-4 rounded-xl shadow-lg hover:bg-slate-800 transition-colors"
        >
          Open Camera
        </button>
      </div>
    {/if}

    <!-- PAYMENT DETAILS -->
    {#if isLoggedIn && qrCode}
      <div class="flex flex-col gap-5" in:slide={{ duration: 400, axis: "y" }}>
        <!-- Spot Info -->
        <div
          class="bg-white border border-slate-100 p-4 rounded-2xl flex items-center justify-between shadow-sm"
        >
          <div class="flex items-center gap-3">
            <div
              class="w-10 h-10 bg-orange-50 rounded-lg flex items-center justify-center text-orange-600"
            >
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
                class="text-xs text-slate-500 font-semibold uppercase tracking-wider"
              >
                Spot ID
              </p>
              <p class="text-lg font-bold text-slate-900 font-mono">{qrCode}</p>
            </div>
          </div>
          <button
            class="text-sm font-medium text-indigo-600 hover:text-indigo-700 px-3 py-1.5 bg-indigo-50 rounded-lg transition-colors"
            on:click={() => (qrCode = "")}
          >
            Change
          </button>
        </div>

        <!-- Calculator -->
        <div
          class="bg-white p-6 rounded-3xl shadow-[0_8px_30px_rgb(0,0,0,0.04)] border border-slate-100 flex flex-col gap-6"
        >
          <div class="flex flex-col gap-2">
            <label class="text-sm font-bold text-slate-700 ml-1"
              >How many hours?</label
            >
            <div class="relative group">
              <input
                type="number"
                min="0"
                bind:value={hours}
                class="w-full bg-slate-50 border-2 border-slate-100 focus:border-indigo-500 px-5 py-4 rounded-xl text-2xl font-bold text-slate-900 outline-none transition-colors text-center placeholder-slate-300"
                placeholder="0"
              />
              <div
                class="absolute right-5 top-1/2 -translate-y-1/2 text-slate-400 font-medium pointer-events-none text-sm"
              >
                HR
              </div>
            </div>
          </div>

          <div class="bg-indigo-50/50 rounded-2xl p-5 border border-indigo-100">
            <div class="flex justify-between items-end">
              <span class="text-slate-600 font-medium">Total Amount</span>
              <div class="text-right">
                <span class="text-3xl font-bold text-slate-900"
                  >{cost.toLocaleString()}</span
                >
                <span class="text-lg font-semibold text-slate-500 ml-1"
                  >IQD</span
                >
              </div>
            </div>
            <div
              class="mt-2 text-xs font-semibold text-indigo-600 flex items-center justify-end gap-1"
            >
              <span class="w-1.5 h-1.5 rounded-full bg-indigo-600"></span>
              Rate: 1,000 IQD / Hour
            </div>
          </div>

          <button
            on:click={pay}
            disabled={hours <= 0}
            class="w-full py-4 rounded-xl bg-indigo-600 text-white font-bold shadow-lg shadow-indigo-200 hover:bg-indigo-700 hover:shadow-xl hover:scale-[1.01] active:scale-[0.98] transition-all disabled:opacity-50 disabled:cursor-not-allowed disabled:shadow-none"
          >
            Confirm & Pay
          </button>
        </div>
      </div>
    {/if}
  </main>

  {#if authCode}
    <div class="py-4 text-center opacity-40">
      <code class="text-[10px] text-slate-400 font-mono tracking-widest"
        >{authCode.slice(0, 8)}••••••••</code
      >
    </div>
  {/if}
</div>

<style>
  :global(body) {
    background-color: #fafafa;
  }
</style>
