<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>স্মার্ট হিসাব খাতা ২০২৬ - ফাইনাল ফিক্স</title>
    
    <link href="https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@400;600;700&display=swap" rel="stylesheet">

    <style>
        * { box-sizing: border-box; }
        body { font-family: 'Hind Siliguri', sans-serif; background-color: #f0f4f8; padding: 10px; margin: 0; }
        .container { max-width: 800px; margin: auto; background: #fff; padding: 15px; border-radius: 8px; }
        
        .input-box { background: #e8f0fe; padding: 15px; border-radius: 8px; margin-bottom: 20px; border: 1px solid #bcdefa; }
        input, button { width: 100%; padding: 12px; margin: 5px 0; border: 1px solid #ccc; border-radius: 6px; font-family: 'Hind Siliguri'; font-size: 16px; font-weight: bold; }
        
        .btn-save { background: #f39c12; color: white; border: none; cursor: pointer; }
        .btn-pdf { background: #27ae60; color: white; border: none; cursor: pointer; margin-top: 5px; }
        .btn-clear { background: #607d8b; color: white; border: none; margin-top: 20px; cursor: pointer; font-size: 14px; }

        .card-table { width: 100%; border-collapse: collapse; margin-bottom: 30px; background-color: #fff; border: 2px solid #000; }
        .card-table td { 
            border: 2px solid #000; 
            padding: 10px; 
            text-align: center; 
            font-size: 19px; 
            font-weight: 700 !important; 
            color: #000 !important; 
        }

        .btn-del-last { 
            background: #ff4d4d; color: white; border: none; border-radius: 4px;
            cursor: pointer; font-size: 12px; padding: 2px 8px; margin-top: 5px;
            font-weight: normal; display: block; margin-left: auto; margin-right: auto;
        }

        .bg-blue { background-color: #d1e7ff !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
        .bg-yellow { background-color: #ffff00 !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
        .bg-orange { background-color: #fcd5b4 !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
        .bg-light-yellow { background-color: #fff2cc !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
        .bg-total-dep { background-color: #c5e1a5 !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }

        .editable { cursor: text; outline: none; }
        .editable:focus { background-color: #fff9c4 !important; }

        @media print {
            .input-box, .btn-clear, .btn-del-last, h3 { display: none !important; }
        }
    </style>
</head>
<body>

<div class="container">
    <h3 style="text-align:center;">স্মার্ট হিসাব খাতা ২০২৬</h3>

    <div class="input-box">
        <label>তারিখ নির্বাচন:</label>
        <input type="date" id="tDate">
        <input type="number" id="tDeposit" placeholder="জমার পরিমাণ">
        <hr>
        <input type="text" id="tDesc" placeholder="খরচের বিবরণ (উদ্দেশ্য)">
        <input type="number" id="tAmount" placeholder="খরচের পরিমাণ">
        
        <button class="btn-save" onclick="saveToCard()">হিসাব সেভ করুন</button>
        <button class="btn-pdf" onclick="window.print()">পিডিএফ ডাউনলোড করুন</button>
    </div>

    <div id="cardDisplay"></div>
    <button class="btn-clear" onclick="clearAllData()">সব ডাটা মুছে ফেলুন</button>
</div>

<script>
    function toBanglaNum(n) {
        if(n === undefined || n === null) return "০";
        const b = {'0':'০','1':'১','2':'২','3':'৩','4':'৪','5':'৫','6':'৬','7':'৭','8':'৮','9':'৯'};
        return n.toString().replace(/[0-9]/g, w => b[w]);
    }

    function toBanglaDate(d) {
        if(!d) return "";
        const p = d.split('-');
        return toBanglaNum(p[2]) + "/" + toBanglaNum(p[1]) + "/" + toBanglaNum(p[0]);
    }

    window.onload = function() {
        document.getElementById('tDate').valueAsDate = new Date();
        loadCards();
    };

    function saveToCard() {
        const date = document.getElementById('tDate').value;
        const dep = parseFloat(document.getElementById('tDeposit').value) || 0;
        const desc = document.getElementById('tDesc').value;
        const amt = parseFloat(document.getElementById('tAmount').value) || 0;

        if (!date) return alert("তারিখ দিন");

        let saved = JSON.parse(localStorage.getItem('Hishab_V_Final_Bold')) || [];
        if (saved.length === 0) saved.push({ deposits: [], exps: [], totalDep: 0, totalExp: 0 });

        let card = saved[0];

        if (dep > 0) {
            card.deposits.push({ amount: dep, date: date });
            card.totalDep += dep;
        }

        if (desc && amt > 0) {
            card.exps.push({ desc: desc, amt: amt, date: date });
            card.totalExp += amt;
        }

        localStorage.setItem('Hishab_V_Final_Bold', JSON.stringify(saved));
        
        document.getElementById('tDeposit').value = "";
        document.getElementById('tDesc').value = "";
        document.getElementById('tAmount').value = "";
        loadCards();
    }

    // ডিলিট ফাংশন সংশোধন করা হয়েছে
    function deleteLastRow(type) {
        let saved = JSON.parse(localStorage.getItem('Hishab_V_Final_Bold')) || [];
        if (saved.length > 0) {
            let card = saved[0];
            if (type === 'dep' && card.deposits.length > 0) {
                let lastItem = card.deposits.pop();
                card.totalDep -= lastItem.amount;
            } else if (type === 'exp' && card.exps.length > 0) {
                let lastItem = card.exps.pop();
                card.totalExp -= lastItem.amt; // এখানে ভুল ছিল, ঠিক করা হয়েছে
            }
            localStorage.setItem('Hishab_V_Final_Bold', JSON.stringify(saved));
            loadCards();
        }
    }

    function loadCards() {
        let saved = JSON.parse(localStorage.getItem('Hishab_V_Final_Bold')) || [];
        const display = document.getElementById('cardDisplay');
        display.innerHTML = "";

        saved.forEach(data => {
            const balance = data.totalDep - data.totalExp;
            let tableHTML = `<table class="card-table">`;

            // জমার অংশ
            data.deposits.forEach((d, index) => {
                const isLast = (index === data.deposits.length - 1);
                tableHTML += `<tr class="bg-blue">
                    <td class="editable" contenteditable="true">${toBanglaDate(d.date)}</td>
                    <td class="editable" contenteditable="true">জমা</td>
                    <td class="editable" contenteditable="true">
                        ${toBanglaNum(d.amount)}/-
                        ${isLast ? `<button class="btn-del-last" onclick="deleteLastRow('dep')">মুছুন</button>` : ''}
                    </td>
                    <td class="editable" contenteditable="true"></td>
                </tr>`;
            });

            tableHTML += `<tr class="bg-total-dep">
                <td colspan="2" style="text-align:right" class="editable" contenteditable="true">মোট জমা:</td>
                <td colspan="2" class="editable" contenteditable="true">${toBanglaNum(data.totalDep)}/- টাকা</td>
            </tr>`;

            tableHTML += `<tr class="bg-yellow">
                <td class="editable" contenteditable="true">তারিখ</td>
                <td class="editable" contenteditable="true">উদ্দেশ্য</td>
                <td class="editable" contenteditable="true">খরচ</td>
                <td class="editable" contenteditable="true">মন্তব্য</td>
            </tr>`;

            // খরচের অংশ
            data.exps.forEach((e, index) => {
                const isLast = (index === data.exps.length - 1);
                tableHTML += `<tr>
                    <td class="editable" contenteditable="true">${toBanglaDate(e.date)}</td>
                    <td style="text-align:left" class="editable" contenteditable="true">${e.desc}</td>
                    <td class="editable" contenteditable="true">
                        ${toBanglaNum(e.amt)}/-
                        ${isLast ? `<button class="btn-del-last" onclick="deleteLastRow('exp')">মুছুন</button>` : ''}
                    </td>
                    <td class="editable" contenteditable="true"></td>
                </tr>`;
            });

            if(data.exps.length === 0) {
                tableHTML += `<tr><td colspan="4" class="editable" contenteditable="true">কোনো খরচ নেই</td></tr>`;
            }

            // মোট খরচ ও অবশিষ্ট আপডেট
            tableHTML += `<tr class="bg-orange">
                <td class="editable" contenteditable="true"></td>
                <td style="text-align:right" class="editable" contenteditable="true">মোট খরচ: ${toBanglaNum(data.totalExp)}/-</td>
                <td colspan="2" class="bg-light-yellow editable" contenteditable="true">
                    অবশিষ্ট: ${toBanglaNum(balance)}/- টাকা
                </td>
            </tr></table>`;

            display.innerHTML += tableHTML;
        });
    }

    function clearAllData() {
        if(confirm("সব ডাটা মুছে যাবে!")) {
            localStorage.removeItem('Hishab_V_Final_Bold');
            location.reload();
        }
    }
</script>

</body>
</html>
