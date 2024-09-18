# Earning Rewards
Sequoia providers will earn storage rewards over time proportional to how much storage they are providing.

## Jackal Storage Provider Profitability Calculator
This is just an estimate, there is no guarantee you will receive this amount.


<form id="checker">
    <label for="amount">TBs Offered:</label>
    <input type="number" id="amount" name="amount" value="2"><br><br>
    <label for="usage-ratio">Usage Percentage:</label>
    <input type="range" min="1" max="100" value="33" class="slider" id="usage-ratio" oninput="this.nextElementSibling.value = this.value + `%`">
    <output>33%</output>
</form>
    
<br>
<span>$ <span id="result">0</span> / month</span>

<script>
    
        window.addEventListener("load", start);
    
        const price = 15;
        const networkSize = 200;
    
        function start() {
            document.getElementById("amount").addEventListener("change", updateResult);
            document.getElementById("usage-ratio").addEventListener("change", updateResult);
            updateResult()
        }
    
        function updateResult() {
            const amount = document.getElementById("amount").value;
            const ratio = document.getElementById("usage-ratio").value;
    
            const networkProfit = networkSize * price;
    
            const ownership = Number(amount)/ networkSize;
    
    
            const myProfit = ownership * networkProfit * (ratio / 100);
    
            document.getElementById("result").innerText = myProfit.toFixed(2).toString();
        }
    
    
</script>
