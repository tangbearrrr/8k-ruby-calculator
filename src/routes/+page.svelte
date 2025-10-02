<script lang="ts">
    let dailyKeys = '';
    let showBox20 = false;
    let showBox50 = false;
    let selectedArea: keyof typeof areaRounds | '' = '';
    
    let totalKeys = 0;
    let rounds = 0;
    let sets = 0;
    let grossRubies = 0;
    let rubiesSpent = 0;
    let netRubies = 0;
    let remainingKeys = 0;

    const areaRounds = {
        '1': 18,
        '2-7': 13,
        '8': 12,
        '9': 12,
        '10': 12,
        '11': 11,
        '12': 11,
        '13': 11,
        '14': 10,
        '15+': 10,
        'NM': 10
    } as const;

    $: dailyKeysCount = Math.floor(parseInt(dailyKeys, 10) || 0);
    $: box20Count = showBox20 ? 20 : 0;
    $: box50Count = showBox50 ? 50 : 0;
    $: roundsPerSet = selectedArea ? areaRounds[selectedArea as keyof typeof areaRounds] : 0;
    $: keysPerRound = selectedArea === 'NM' ? 12 : 6;
    $: rubiesPerSet = selectedArea === 'NM' ? 160 : 80;

    $: keySourceData = {
        daily: {
            keys: dailyKeysCount,
            rounds: Math.floor(dailyKeysCount / keysPerRound),
            sets: roundsPerSet > 0 ? Math.floor(Math.floor(dailyKeysCount / keysPerRound) / roundsPerSet) : 0,
            rubiesEarned: roundsPerSet > 0 ? Math.floor(Math.floor(Math.floor(dailyKeysCount / keysPerRound) / roundsPerSet) * rubiesPerSet) : 0,
            rubiesSpent: 0,
            netGain: roundsPerSet > 0 ? Math.floor(Math.floor(Math.floor(dailyKeysCount / keysPerRound) / roundsPerSet) * rubiesPerSet) : 0
        },
        box20: {
            keys: box20Count * 60,
            rounds: Math.floor((box20Count * 60) / keysPerRound),
            sets: roundsPerSet > 0 ? Math.floor(Math.floor((box20Count * 60) / keysPerRound) / roundsPerSet) : 0,
            rubiesEarned: roundsPerSet > 0 ? Math.floor(Math.floor(Math.floor((box20Count * 60) / keysPerRound) / roundsPerSet) * rubiesPerSet) : 0,
            rubiesSpent: box20Count * 50,
            netGain: roundsPerSet > 0 ? Math.floor(Math.floor(Math.floor((box20Count * 60) / keysPerRound) / roundsPerSet) * rubiesPerSet) - (box20Count * 50) : -(box20Count * 50)
        },
        box50: {
            keys: box50Count * 60,
            rounds: Math.floor((box50Count * 60) / keysPerRound),
            sets: roundsPerSet > 0 ? Math.floor(Math.floor((box50Count * 60) / keysPerRound) / roundsPerSet) : 0,
            rubiesEarned: roundsPerSet > 0 ? Math.floor(Math.floor(Math.floor((box50Count * 60) / keysPerRound) / roundsPerSet) * rubiesPerSet) : 0,
            rubiesSpent: box50Count * 80,
            netGain: roundsPerSet > 0 ? Math.floor(Math.floor(Math.floor((box50Count * 60) / keysPerRound) / roundsPerSet) * rubiesPerSet) - (box50Count * 80) : -(box50Count * 80)
        }
    };

    $: {
        totalKeys = keySourceData.daily.keys + keySourceData.box20.keys + keySourceData.box50.keys;
        rounds = Math.floor(totalKeys / keysPerRound);
        sets = roundsPerSet > 0 ? Math.floor(rounds / roundsPerSet) : 0;
        remainingKeys = totalKeys - (rounds * keysPerRound);
        grossRubies = sets * rubiesPerSet;
        rubiesSpent = keySourceData.box20.rubiesSpent + keySourceData.box50.rubiesSpent;
        netRubies = grossRubies - rubiesSpent;
    }

    $: activeKeySources = [
        dailyKeysCount > 0 && { name: 'Daily Keys', ...keySourceData.daily },
        showBox20 && { name: '20 Key Boxes', ...keySourceData.box20 },
        showBox50 && { name: '50 Key Boxes', ...keySourceData.box50 }
    ].filter(Boolean);
</script>

<style>
    :global(body) {
        background: linear-gradient(135deg, 
            #1a1a2e 0%, 
            #16213e 25%, 
            #0f3460 50%, 
            #16213e 75%, 
            #1a1a2e 100%);
        background-size: 400% 400%;
        animation: gradientShift 15s ease infinite;
        min-height: 100vh;
        margin: 0;
        font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'Helvetica Neue', Arial, sans-serif;
        overflow-x: hidden;
    }

    @keyframes gradientShift {
        0% { background-position: 0% 50%; }
        50% { background-position: 100% 50%; }
        100% { background-position: 0% 50%; }
    }

    .container {
        max-width: 800px;
        margin: 4rem auto 0 auto;
        display: flex;
        gap: 1.5rem;
        padding: 0 1rem;
        position: relative;
    }

    .simple-window {
        background: rgba(30, 30, 50, 0.3);
        backdrop-filter: blur(20px);
        -webkit-backdrop-filter: blur(20px);
        flex: 1;
        border-radius: 24px;
        border: 1px solid rgba(100, 120, 180, 0.2);
        box-shadow: 
            0 8px 32px rgba(0, 0, 0, 0.3),
            inset 0 1px 0 rgba(100, 120, 180, 0.3),
            inset 0 -1px 0 rgba(100, 120, 180, 0.1);
        padding: 2rem 1.5rem 1.5rem 1.5rem;
        color: rgba(220, 220, 240, 0.95);
        position: relative;
        overflow: hidden;
        transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
    }

    .simple-window::before {
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        height: 1px;
        background: linear-gradient(90deg, 
            transparent 0%, 
            rgba(100, 120, 180, 0.6) 50%, 
            transparent 100%);
    }

    .simple-window:hover {
        transform: translateY(-2px);
        box-shadow: 
            0 12px 40px rgba(0, 0, 0, 0.4),
            inset 0 1px 0 rgba(100, 120, 180, 0.4),
            inset 0 -1px 0 rgba(100, 120, 180, 0.2);
    }

    .results-container {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        width: 280px;
        height: fit-content;
        position: sticky;
        top: 4rem;
    }

    .result-box {
        background: rgba(30, 30, 50, 0.3);
        backdrop-filter: blur(20px);
        -webkit-backdrop-filter: blur(20px);
        border-radius: 20px;
        border: 1px solid rgba(100, 120, 180, 0.2);
        box-shadow: 
            0 6px 24px rgba(0, 0, 0, 0.3),
            inset 0 1px 0 rgba(100, 120, 180, 0.3),
            inset 0 -1px 0 rgba(100, 120, 180, 0.1);
        padding: 1.25rem;
        color: rgba(220, 220, 240, 0.95);
        overflow: hidden;
        transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        position: relative;
    }

    .result-box::before {
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        height: 1px;
        background: linear-gradient(90deg, 
            transparent 0%, 
            rgba(100, 120, 180, 0.6) 50%, 
            transparent 100%);
    }

    .result-box:hover {
        transform: translateY(-2px);
        box-shadow: 
            0 10px 32px rgba(0, 0, 0, 0.4),
            inset 0 1px 0 rgba(100, 120, 180, 0.4),
            inset 0 -1px 0 rgba(100, 120, 180, 0.2);
    }

    .simple-header {
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 0.6rem;
        font-size: 1.3rem;
        font-weight: 700;
        color: rgba(220, 220, 240, 0.95);
        margin-bottom: 1.2rem;
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
    }

    .ruby-icon {
        width: 2rem;
        vertical-align: middle;
        filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.4));
        animation: float 3s ease-in-out infinite;
    }

    .warning-message {
        background: rgba(255, 107, 138, 0.2);
        border: 1px solid rgba(255, 107, 138, 0.4);
        border-radius: 12px;
        padding: 0.8rem 1rem;
        margin-bottom: 1.5rem;
        color: #ff6b8a;
        font-weight: 600;
        font-size: 0.95rem;
        text-align: center;
        backdrop-filter: blur(10px);
        -webkit-backdrop-filter: blur(10px);
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
        animation: pulse 2s ease-in-out infinite;
    }

    @keyframes pulse {
        0%, 100% { opacity: 1; }
        50% { opacity: 0.8; }
    }

    label {
        font-size: 1rem;
        font-weight: 600;
        margin-bottom: 0.3rem;
        display: block;
        color: rgba(200, 200, 220, 0.9);
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
    }

    input[type="number"] {
        margin-top: 0.3rem;
        width: 100%;
        box-sizing: border-box;
        padding: 0.8rem 1rem;
        border-radius: 16px;
        border: 1px solid rgba(100, 120, 180, 0.3);
        background: rgba(20, 20, 40, 0.4);
        backdrop-filter: blur(10px);
        -webkit-backdrop-filter: blur(10px);
        color: rgba(220, 220, 240, 0.95);
        font-size: 1rem;
        margin-bottom: 1rem;
        outline: none;
        transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        font-family: inherit;
    }

    input[type="number"]::placeholder {
        color: rgba(150, 150, 180, 0.6);
    }

    input[type="number"]:focus {
        border: 1px solid rgba(100, 120, 180, 0.6);
        background: rgba(20, 20, 40, 0.6);
        transform: scale(1.02);
        box-shadow: 
            0 0 0 4px rgba(100, 120, 180, 0.2),
            inset 0 1px 0 rgba(100, 120, 180, 0.3);
    }

    select {
        margin-top: 0.3rem;
        width: 100%;
        box-sizing: border-box;
        padding: 0.8rem 1rem;
        border-radius: 16px;
        border: 1px solid rgba(100, 120, 180, 0.3);
        background: rgba(20, 20, 40, 0.4);
        backdrop-filter: blur(10px);
        -webkit-backdrop-filter: blur(10px);
        color: rgba(220, 220, 240, 0.95);
        font-size: 1rem;
        margin-bottom: 1rem;
        outline: none;
        transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        cursor: pointer;
        font-family: inherit;
        appearance: none;
        background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='rgba(150,150,180,0.8)' viewBox='0 0 16 16'%3e%3cpath d='m7.247 4.86-4.796 5.481c-.566.647-.106 1.659.753 1.659h9.592a1 1 0 0 0 .753-1.659l-4.796-5.48a1 1 0 0 0-1.506 0z'/%3e%3c/svg%3e");
        background-repeat: no-repeat;
        background-position: right 0.75rem center;
        background-size: 16px 12px;
        padding-right: 2.5rem;
    }

    select option {
        background: rgba(15, 15, 30, 0.95);
        color: rgba(220, 220, 240, 0.95);
        padding: 0.5rem;
    }

    select:focus {
        border: 1px solid rgba(100, 120, 180, 0.6);
        background: rgba(20, 20, 40, 0.6);
        transform: scale(1.02);
        box-shadow: 
            0 0 0 4px rgba(100, 120, 180, 0.2),
            inset 0 1px 0 rgba(100, 120, 180, 0.3);
    }

    .section-header {
        margin-bottom: 0.5rem;
        font-weight: 700;
        color: rgba(220, 220, 240, 0.95);
        font-size: 1.1rem;
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
    }

    .stat-row {
        margin-bottom: 0.3rem;
        font-weight: 500;
    }

    .stat-total {
        font-size: 1.1rem;
        margin-top: 0.5rem;
        padding-top: 0.5rem;
        border-top: 1px solid rgba(100, 120, 180, 0.3);
        font-weight: 600;
    }

    .negative-value {
        color: #ff6b8a;
        font-weight: 700;
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.4);
    }

    .total-value {
        font-size: 1.2rem;
        font-weight: 700;
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.4);
    }

    .ruby-highlight {
        color: #64b5f6;
        font-weight: 700;
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.4);
    }

    .checkbox-container {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        margin-bottom: 0.5rem;
        padding: 0.5rem 0;
    }

    input[type="checkbox"] {
        width: 20px;
        height: 20px;
        margin: 0;
        appearance: none;
        border: 2px solid rgba(100, 120, 180, 0.4);
        border-radius: 6px;
        background: rgba(20, 20, 40, 0.4);
        backdrop-filter: blur(10px);
        -webkit-backdrop-filter: blur(10px);
        cursor: pointer;
        transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        position: relative;
    }

    input[type="checkbox"]:checked {
        background: rgba(100, 120, 180, 0.3);
        border-color: rgba(100, 120, 180, 0.7);
        transform: scale(1.1);
    }

    input[type="checkbox"]:checked::after {
        content: '✓';
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        color: rgba(220, 220, 240, 0.95);
        font-size: 12px;
        font-weight: bold;
        text-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
    }

    input[type="checkbox"]:hover {
        transform: scale(1.05);
        border-color: rgba(100, 120, 180, 0.6);
    }

    .checkbox-label {
        margin: 0;
        font-weight: 600;
        color: rgba(200, 200, 220, 0.9);
        cursor: pointer;
        transition: color 0.3s ease;
    }

    .checkbox-label:hover {
        color: rgba(220, 220, 240, 1);
    }

    .placeholder {
        color: rgba(120, 120, 150, 0.6);
        font-style: italic;
    }

    /* Floating animation for icons */
    .ruby-icon {
        animation: float 3s ease-in-out infinite;
    }

    @keyframes float {
        0%, 100% { transform: translateY(0px); }
        50% { transform: translateY(-3px); }
    }

    .credit-footer {
        text-align: center;
        margin-top: 3rem;
        margin-bottom: 2rem;
        color: rgba(150, 150, 180, 0.7);
        font-size: 0.9rem;
        font-weight: 500;
    }

    .contact-info {
        margin-top: 0.5rem;
        font-size: 0.85rem;
    }

    .contact-info a {
        color: rgba(100, 120, 180, 0.8);
        text-decoration: none;
        transition: color 0.3s ease;
    }

    .contact-info a:hover {
        color: rgba(100, 120, 180, 1);
        text-decoration: underline;
    }

    /* Responsive design for mobile */
    @media (max-width: 768px) {
        .container {
            flex-direction: column;
            gap: 1rem;
            margin-top: 2rem;
        }
        
        .results-container {
            width: 100%;
            position: static;
        }
        
        .simple-window, .result-box {
            border-radius: 20px;
            padding: 1.5rem 1rem;
        }
    }
</style>

<div class="container">
    <div class="simple-window">
        <div class="simple-header">
            <img class="ruby-icon" src="https://skiagb.gcdn.netmarble.com/skiagb/web_images/product/detail/1/11.png" alt="Ruby" />
            Ruby Calculator
        </div>
        
        <!-- Warning message -->
        <div class="warning-message">
            ⚠️ Now supports only Premium
        </div>
        
        <label>
            Daily Keys Received
            <input type="number" bind:value={dailyKeys} min="0" placeholder="Enter daily keys..." />
        </label>

        <div class="checkbox-container">
            <input type="checkbox" id="box20-check" bind:checked={showBox20} />
            <label for="box20-check" class="checkbox-label">20 Key Boxes (50 rubies each)</label>
        </div>

        <div class="checkbox-container">
            <input type="checkbox" id="box50-check" bind:checked={showBox50} />
            <label for="box50-check" class="checkbox-label">50 Key Boxes (80 rubies each)</label>
        </div>

        <label>
            Select Area
            <select bind:value={selectedArea}>
                <option value="">Please select area</option>
                <option value="1">Area 1</option>
                <option value="2-7">Area 2-7</option>
                <option value="8">Area 8</option>
                <option value="9">Area 9</option>
                <option value="10">Area 10</option>
                <option value="11">Area 11</option>
                <option value="12">Area 12</option>
                <option value="13">Area 13</option>
                <option value="14">Area 14</option>
                <option value="15+">Area 15+</option>
                <option value="NM">Nightmare</option>
            </select>
            {#if selectedArea}
            <div style="font-size: 0.85rem; color: #6b7685; margin-top: -0.5rem; margin-bottom: 1rem;">
                Rounds per Set: {areaRounds[selectedArea as keyof typeof areaRounds]}
            </div>
            {/if}
        </label>
    </div>

    <div class="results-container">
        <!-- Farming Summary Box -->
        <div class="result-box">
            <div class="section-header">
                Farming Summary
            </div>
            <div class="stat-row">
                Total Keys: <span class="{totalKeys > 0 ? 'ruby-highlight' : 'placeholder'}">{totalKeys > 0 ? totalKeys : '-'}</span>
            </div>
            <div class="stat-row">
                Farming Rounds: <span class="{rounds > 0 ? 'ruby-highlight' : 'placeholder'}">{rounds > 0 ? rounds : '-'}</span>
            </div>
            <div class="stat-row">
                Complete Sets: <span class="{sets > 0 ? 'ruby-highlight' : 'placeholder'}">{sets > 0 ? sets : '-'}</span>
            </div>
            <div class="stat-row">
                Remaining Keys: <span class="{totalKeys > 0 ? 'ruby-highlight' : 'placeholder'}">{totalKeys > 0 ? remainingKeys : '-'}</span>
            </div>
        </div>
        
        <!-- Ruby Spend Box -->
        <div class="result-box">
            <div class="section-header">
                Ruby Spend
            </div>
            {#if showBox20}
            <div class="stat-row">
                20 Key Boxes: <span class="negative-value">-{keySourceData.box20.rubiesSpent}</span>
            </div>
            {:else}
            <div class="stat-row">
                20 Key Boxes: <span class="placeholder">-</span>
            </div>
            {/if}
            {#if showBox50}
            <div class="stat-row">
                50 Key Boxes: <span class="negative-value">-{keySourceData.box50.rubiesSpent}</span>
            </div>
            {:else}
            <div class="stat-row">
                50 Key Boxes: <span class="placeholder">-</span>
            </div>
            {/if}
            <div class="stat-total">
                Total Ruby Spend: <span class="{rubiesSpent > 0 ? 'negative-value total-value' : 'placeholder total-value'}">{rubiesSpent > 0 ? `-${rubiesSpent}` : '-'}</span>
            </div>
        </div>
        
        <!-- Net Gain Box -->
        <div class="result-box">
            <div class="section-header">
                Net Gain
            </div>
            <div class="stat-total">
                Total Net Rubies: <span class="{totalKeys > 0 ? 'ruby-highlight total-value' : 'placeholder total-value'}" style="color: {totalKeys > 0 ? (netRubies >= 0 ? '#28a745' : '#dc3545') : 'inherit'};">{totalKeys > 0 ? (netRubies >= 0 ? '+' : '') + netRubies : '-'}</span>
            </div>
        </div>
    </div>
</div>

<!-- Credit footer -->
<div class="credit-footer">
    Need supports ?
    <div class="contact-info">
        Contact us: <a href="mailto:mheenotalk@gmail.com">mheenotalk@gmail.com</a>
    </div>
</div>