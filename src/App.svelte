<script>
  let activeMenu = 'clo'; // 'ips', 'ipk', 'clo'

  const bobotNilai = {
    'A': 4.0, 'AB': 3.5, 'B': 3.0, 'BC': 2.5, 'C': 2.0, 'D': 1.0, 'E': 0.0
  };

  const batasNilai = {
    'A': '> 85.00', 'AB': '> 75.00', 'B': '> 65.00', 'BC': '> 55.00',
    'C': '> 50.00', 'D': '> 40.00', 'E': '<= 40.00'
  };

  let mataKuliahList = [
    { nama: '', sks: 0, nilai: 'E' },
  ];

  function tambahBarisMk() { mataKuliahList = [...mataKuliahList, { nama: '', sks: 0, nilai: 'E' }]; }
  function hapusBarisMk(index) { mataKuliahList = mataKuliahList.filter((_, i) => i !== index); }

  $: totalSksIps = mataKuliahList.reduce((sum, mk) => sum + (Number(mk.sks) || 0), 0);
  $: totalMutuIps = mataKuliahList.reduce((sum, mk) => sum + ((bobotNilai[mk.nilai] || 0) * (Number(mk.sks) || 0)), 0);
  $: ips = totalSksIps > 0 ? (totalMutuIps / totalSksIps).toFixed(2) : '0.00';

  let semesterList = [
    { nama: 'Semester 1', sks: 0, ips: 0.00 },
  ];

  function tambahBarisSemester() { semesterList = [...semesterList, { nama: `Semester ${semesterList.length + 1}`, sks: 0, ips: 0.00 }]; }
  function hapusBarisSemester(index) { semesterList = semesterList.filter((_, i) => i !== index); }

  $: totalSksIpk = semesterList.reduce((sum, sem) => sum + (Number(sem.sks) || 0), 0);
  $: totalMutuIpk = semesterList.reduce((sum, sem) => sum + ((Number(sem.ips) || 0) * (Number(sem.sks) || 0)), 0);
  $: ipk = totalSksIpk > 0 ? (totalMutuIpk / totalSksIpk).toFixed(2) : '0.00';


  let cloList = [
    {
      nama: 'Sub-CLO-2-1',
      komponen: [
        { nama: 'Kuis', bobot: 0, nilaiList: [{ val: 0 }, { val: 0 }] },
        { nama: 'Tugas', bobot: 0, nilaiList: [{ val: 0 }] },
        { nama: 'Ujian', bobot: 0, nilaiList: [{ val: 0 }] }
      ]
    }
  ];

  function tambahClo() {
    cloList = [...cloList, { nama: `Sub-CLO-${cloList.length + 1}`, komponen: [{ nama: 'Kuis', bobot: 5, nilaiList: [{ val: 0 }] }] }];
  }
  function hapusClo(cIndex) { cloList = cloList.filter((_, i) => i !== cIndex); }
  function tambahKomponen(cIndex) {
    cloList[cIndex].komponen = [...cloList[cIndex].komponen, { nama: 'Tugas', bobot: 5, nilaiList: [{ val: 0 }] }];
  }
  function hapusKomponen(cIndex, kIndex) {
    cloList[cIndex].komponen = cloList[cIndex].komponen.filter((_, i) => i !== kIndex);
  }
  function tambahNilai(cIndex, kIndex) {
    cloList[cIndex].komponen[kIndex].nilaiList = [...cloList[cIndex].komponen[kIndex].nilaiList, { val: 0 }];
  }
  function hapusNilai(cIndex, kIndex, nIndex) {
    cloList[cIndex].komponen[kIndex].nilaiList = cloList[cIndex].komponen[kIndex].nilaiList.filter((_, i) => i !== nIndex);
  }

  $: hitungNilaiKomponen = (komponen) => {
    if (komponen.nilaiList.length === 0) return 0;
    const total = komponen.nilaiList.reduce((sum, n) => sum + (Number(n.val) || 0), 0);
    const rataRata = total / komponen.nilaiList.length;
    return rataRata * ((Number(komponen.bobot) || 0) / 100);
  };

  $: hitungTotalCLO = (clo) => clo.komponen.reduce((sum, kom) => sum + hitungNilaiKomponen(kom), 0);

  $: totalBobotClo = cloList.reduce((sum, clo) => 
    sum + clo.komponen.reduce((kSum, kom) => kSum + (Number(kom.bobot) || 0), 0)
  , 0);

  $: totalNilaiAkhirClo = cloList.reduce((sum, clo) => sum + hitungTotalCLO(clo), 0);

  $: hurufPrediksiClo = (() => {
    const n = totalNilaiAkhirClo;
    if (n > 85) return 'A';
    if (n > 75) return 'AB';
    if (n > 65) return 'B';
    if (n > 55) return 'BC';
    if (n > 50) return 'C';
    if (n > 40) return 'D';
    return 'E';
  })();
</script>

<main class="min-h-screen bg-slate-50 py-10 px-4 sm:px-6 lg:px-8 font-sans">
  <div class="max-w-4xl mx-auto bg-white rounded-2xl shadow-xl overflow-hidden border border-slate-100">
    
    <div class="bg-gradient-to-r from-indigo-600 to-violet-600 p-6 sm:p-8 text-white text-center">
      <h1 class="text-3xl font-extrabold tracking-tight">CekIPCuy</h1>
      <p class="mt-2 text-indigo-100 text-sm sm:text-base">Hitung IPs, IPK, dan Simulasi Nilai, biar IPKnya ga 2.3</p>
    </div>

    <div class="flex border-b border-slate-200 overflow-x-auto">
      <button 
        class="flex-1 py-4 px-4 text-sm font-bold text-center whitespace-nowrap transition-colors {activeMenu === 'ips' ? 'text-indigo-600 border-b-2 border-indigo-600 bg-indigo-50/50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-50'}"
        on:click={() => activeMenu = 'ips'}
      >
        IPs
      </button>
      <button 
        class="flex-1 py-4 px-4 text-sm font-bold text-center whitespace-nowrap transition-colors {activeMenu === 'ipk' ? 'text-violet-600 border-b-2 border-violet-600 bg-violet-50/50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-50'}"
        on:click={() => activeMenu = 'ipk'}
      >
        IPK
      </button>
      <button 
        class="flex-1 py-4 px-4 text-sm font-bold text-center whitespace-nowrap transition-colors {activeMenu === 'clo' ? 'text-sky-600 border-b-2 border-sky-600 bg-sky-50/50' : 'text-slate-500 hover:text-slate-700 hover:bg-slate-50'}"
        on:click={() => activeMenu = 'clo'}
      >
        Simulasi Nilai
      </button>
    </div>

    <div class="p-4 sm:p-8">
      
      {#if activeMenu === 'ips'}
        <div class="space-y-3">
          <div class="hidden sm:grid grid-cols-12 gap-4 text-xs font-bold text-slate-500 uppercase tracking-wider px-2">
            <div class="col-span-6">Mata Kuliah</div>
            <div class="col-span-3 text-center">SKS</div>
            <div class="col-span-2 text-center">Nilai</div>
            <div class="col-span-1"></div>
          </div>

          {#each mataKuliahList as mk, index}
            <div class="grid grid-cols-12 gap-2 sm:gap-4 bg-slate-50 p-3 rounded-xl border border-slate-200 items-center">
              <div class="col-span-12 sm:col-span-6">
                <input type="text" bind:value={mk.nama} placeholder="Contoh: Pemrograman Dasar" class="w-full bg-white border border-slate-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500" />
              </div>
              <div class="col-span-2 sm:col-span-3">
                <input type="number" min="1" bind:value={mk.sks} class="w-full bg-white border border-slate-300 rounded-lg px-3 py-2 text-sm text-center focus:outline-none focus:ring-2 focus:ring-indigo-500" />
              </div>
              <div class="col-span-8 sm:col-span-2 flex flex-col">
                <select bind:value={mk.nilai} class="w-full bg-white border border-slate-300 rounded-lg px-2 py-2 text-sm text-center font-bold text-slate-700 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                  {#each Object.keys(bobotNilai) as huruf} 
                    <option value={huruf}>{huruf} ({batasNilai[huruf]})</option> 
                  {/each}
                </select>
              </div>
              <div class="col-span-2 sm:col-span-1 text-center">
                <button on:click={() => hapusBarisMk(index)} class="text-rose-500 hover:text-rose-700 font-bold">✕</button>
              </div>
            </div>
          {/each}
        </div>
        <button on:click={tambahBarisMk} class="mt-4 w-full sm:w-auto bg-slate-100 text-slate-700 font-semibold text-sm py-2 px-4 rounded-lg border border-slate-300">+ Tambah Mata Kuliah</button>

        <div class="mt-8 grid grid-cols-1 sm:grid-cols-2 gap-4">
          <div class="bg-indigo-50 rounded-xl p-4 border border-indigo-100 text-center"><span class="text-xs font-bold text-indigo-500">Total SKS</span> <div class="text-3xl font-black text-indigo-900">{totalSksIps}</div></div>
          <div class="bg-emerald-50 rounded-xl p-4 border border-emerald-100 text-center"><span class="text-xs font-bold text-emerald-600">Estimasi IPs</span> <div class="text-4xl font-black text-emerald-700">{ips}</div></div>
        </div>

      {:else if activeMenu === 'ipk'}
        <div class="space-y-3">
          <div class="hidden sm:grid grid-cols-12 gap-4 text-xs font-bold text-slate-500 uppercase tracking-wider px-2">
            <div class="col-span-6">Semester</div>
            <div class="col-span-3 text-center">Total SKS</div>
            <div class="col-span-2 text-center">IPs</div>
            <div class="col-span-1"></div>
          </div>

          {#each semesterList as sem, index}
            <div class="grid grid-cols-12 gap-2 sm:gap-4 bg-slate-50 p-3 rounded-xl border border-slate-200 items-center">
              <div class="col-span-12 sm:col-span-6"><input type="text" bind:value={sem.nama} class="w-full bg-white border border-slate-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-violet-500" /></div>
              <div class="col-span-6 sm:col-span-3"><input type="number" min="1" bind:value={sem.sks} class="w-full bg-white border border-slate-300 rounded-lg px-3 py-2 text-sm text-center focus:outline-none focus:ring-2 focus:ring-violet-500" /></div>
              <div class="col-span-4 sm:col-span-2"><input type="number" step="0.01" bind:value={sem.ips} class="w-full bg-white border border-slate-300 rounded-lg px-2 py-2 text-sm text-center font-bold focus:outline-none focus:ring-2 focus:ring-violet-500" /></div>
              <div class="col-span-2 sm:col-span-1 text-center"><button on:click={() => hapusBarisSemester(index)} class="text-rose-500 hover:text-rose-700 font-bold">✕</button></div>
            </div>
          {/each}
        </div>
        <button on:click={tambahBarisSemester} class="mt-4 w-full sm:w-auto bg-slate-100 text-slate-700 font-semibold text-sm py-2 px-4 rounded-lg border border-slate-300">➕ Tambah Semester</button>

        <div class="mt-8 grid grid-cols-1 sm:grid-cols-2 gap-4">
          <div class="bg-violet-50 rounded-xl p-4 border border-violet-100 text-center"><span class="text-xs font-bold text-violet-500">SKS Lulus</span> <div class="text-3xl font-black text-violet-900">{totalSksIpk}</div></div>
          <div class="bg-amber-50 rounded-xl p-4 border border-amber-200 text-center"><span class="text-xs font-bold text-amber-600">IPK Keseluruhan</span> <div class="text-4xl font-black text-amber-700">{ipk}</div></div>
        </div>

      {:else}
        <div class="space-y-6">
          {#each cloList as clo, cIndex}
            <div class="bg-slate-50 border border-slate-200 rounded-2xl p-4 sm:p-5 relative">
              <div class="flex justify-between items-center mb-4 border-b border-slate-200 pb-3">
                <input type="text" bind:value={clo.nama} class="bg-transparent text-lg font-bold text-slate-800 border-b border-transparent hover:border-slate-300 focus:border-sky-500 focus:outline-none py-1 px-2" placeholder="Nama Sub-CLO" />
                <button on:click={() => hapusClo(cIndex)} class="text-xs bg-rose-100 text-rose-600 px-3 py-1 rounded-full font-bold hover:bg-rose-200">Hapus CLO</button>
              </div>

              <div class="space-y-3">
                {#each clo.komponen as kom, kIndex}
                  <div class="bg-white border border-slate-200 rounded-xl p-4 shadow-sm relative">
                    <button on:click={() => hapusKomponen(cIndex, kIndex)} class="absolute top-3 right-3 text-rose-400 hover:text-rose-600 font-bold" title="Hapus Komponen">✕</button>
                    
                    <div class="flex flex-wrap gap-4 items-end">
                      <div class="w-full sm:w-auto">
                        <label class="block text-xs font-bold text-slate-500 mb-1">Nama</label>
                        <input type="text" bind:value={kom.nama} class="w-full sm:w-32 bg-slate-50 border border-slate-300 rounded-lg px-3 py-1.5 text-sm focus:outline-none focus:ring-2 focus:ring-sky-500" placeholder="Kuis/Tugas" />
                      </div>
                      <div class="w-24">
                        <label class="block text-xs font-bold text-slate-500 mb-1">Bobot (%)</label>
                        <input type="number" bind:value={kom.bobot} class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-1.5 text-sm text-center focus:outline-none focus:ring-2 focus:ring-sky-500" />
                      </div>

                      <div class="w-full sm:flex-1 pt-2 sm:pt-0">
                        <label class="block text-xs font-bold text-slate-500 mb-1">Nilai</label>
                        <div class="flex flex-wrap gap-2 items-center">
                          {#each kom.nilaiList as n, nIndex}
                            <div class="relative group">
                              <input type="number" step="0.01" bind:value={n.val} class="w-16 sm:w-20 bg-emerald-50 border border-emerald-200 text-emerald-800 font-bold rounded-lg px-2 py-1.5 text-sm text-center focus:outline-none focus:ring-2 focus:ring-emerald-500" />
                              <button on:click={() => hapusNilai(cIndex, kIndex, nIndex)} class="absolute -top-2 -right-2 bg-rose-500 text-white rounded-full w-5 h-5 flex items-center justify-center text-[10px] font-bold opacity-0 group-hover:opacity-100 transition-opacity">✕</button>
                            </div>
                          {/each}
                          <button on:click={() => tambahNilai(cIndex, kIndex)} class="text-xs bg-sky-100 text-sky-700 font-bold px-3 py-1.5 rounded-lg hover:bg-sky-200 border border-sky-200">+ Nilai</button>
                        </div>
                      </div>
                    </div>
                    
                    <div class="mt-3 text-right text-xs text-slate-400 font-medium">
                      Add: <span class="font-bold text-sky-600">{hitungNilaiKomponen(kom).toFixed(2)}</span> / {kom.bobot} poin
                    </div>
                  </div>
                {/each}
              </div>

              <button on:click={() => tambahKomponen(cIndex)} class="mt-4 text-xs font-bold text-slate-600 bg-slate-200 hover:bg-slate-300 px-4 py-2 rounded-lg transition-colors">+ Tambah Komponen (Kuis/UTS dll)</button>
              
              <div class="absolute bottom-4 right-5 text-sm font-bold text-slate-500">
                Sub-Total CLO: <span class="text-lg text-slate-800">{hitungTotalCLO(clo).toFixed(2)}</span>
              </div>
            </div>
          {/each}
        </div>

        <button on:click={tambahClo} class="mt-6 w-full bg-sky-50 border-2 border-dashed border-sky-300 text-sky-600 font-bold text-sm py-4 rounded-xl hover:bg-sky-100 transition-colors">+ Tambah Sub-CLO Baru</button>

        <div class="mt-10 pt-6 border-t border-slate-200 grid grid-cols-1 sm:grid-cols-3 gap-4">
          <div class="bg-slate-50 rounded-xl p-4 border border-slate-200 text-center">
            <span class="text-xs font-bold uppercase tracking-wider text-slate-500">Total Bobot Input</span>
            <div class="text-2xl font-black text-slate-700 mt-1">{totalBobotClo}<span class="text-sm font-medium">%</span></div>
            {#if totalBobotClo !== 100} <p class="text-[10px] text-rose-500 mt-1">Bobot ideal = 100%</p> {/if}
          </div>
          <div class="bg-sky-50 rounded-xl p-4 border border-sky-200 text-center">
            <span class="text-xs font-bold uppercase tracking-wider text-sky-600">Skor Akhir (0-100)</span>
            <div class="text-3xl font-black text-sky-700 mt-1">{totalNilaiAkhirClo.toFixed(2)}</div>
          </div>
          <div class="bg-emerald-50 rounded-xl p-4 border border-emerald-200 text-center">
            <span class="text-xs font-bold uppercase tracking-wider text-emerald-600">Prediksi Index</span>
            <div class="text-4xl font-black text-emerald-600 mt-1">{hurufPrediksiClo}</div>
          </div>
        </div>

      {/if}

      <p class="mt-8 text-center text-xs text-slate-400">
        Aman aja, datanya ga disimpen di database, nanti juga ilang pas close tab.
      </p>
    </div>
  </div>
</main>