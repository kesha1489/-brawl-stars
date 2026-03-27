 <!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Brawl Codes</title>
    <style>
        body {
            background: #1e3c72;
            font-family: Arial;
            padding: 20px;
        }
        .box {
            background: white;
            border-radius: 20px;
            padding: 20px;
            max-width: 400px;
            margin: 0 auto;
        }
        input, button {
            width: 100%;
            padding: 12px;
            margin: 8px 0;
            font-size: 18px;
            border-radius: 12px;
            border: 1px solid #ccc;
        }
        button {
            background: orange;
            font-weight: bold;
        }
        .result {
            background: #f0f0f0;
            padding: 16px;
            margin-top: 16px;
            text-align: center;
            font-size: 22px;
            font-family: monospace;
        }
    </style>
</head>
<body>
<div class="box">
    <h2>Brawl Codes +N</h2>
    <input type="text" id="code" placeholder="Код X.......">
    <input type="number" id="n" placeholder="N">
    <button onclick="calc(1)">+ N</button>
    <button onclick="calc(-1)">- N</button>
    <div class="result" id="result">—</div>
    <button onclick="copy()">Копировать</button>
    <div id="error" style="color:red; margin-top:10px;"></div>
</div>

<script>
const ALPH = "QWERTYUPASDFGHJKLZCVBNM23456789";

function toLong(h,l){ return (BigInt(h)<<32n) | (BigInt(l)&0xFFFFFFFFn); }

function convert(num){
    if(num<=0n) return "";
    let r="", len=BigInt(ALPH.length), n=BigInt(num);
    while(n>0n){
        let idx=Number(n%len);
        r=ALPH[idx]+r;
        n-=BigInt(idx);
        n/=len;
    }
    return r;
}

function decode(code){
    if(!code.startsWith("X") || code.length!==8) return null;
    let b=code.slice(1), len=ALPH.length, u6=0, u7=0n;
    for(let ch of b){
        let idx=ALPH.indexOf(ch);
        if(idx===-1) return null;
        let u12=u6*len+idx;
        u7 = (toLong(Number(u7), u6) * BigInt(len) + BigInt(idx)) >> 32n;
        u6 = u12;
    }
    let v13 = (toLong(Number(u7), u6) >> 8n);
    let lo = Number(v13 & 0x7FFFFFFFn);
    let hi = u6 & 0xFF;
    return { hi, lo, fullId: toLong(hi, lo) };
}

function encode(hi,lo){
    if(hi>=256) return null;
    let l = toLong(lo>>24, hi | ((lo<<8)&0xFFFFFFFF));
    let c = convert(l);
    return c ? "X"+c : null;
}

function calc(sign){
    let code = document.getElementById('code').value.trim().toUpperCase();
    let n = parseInt(document.getElementById('n').value, 10);
    let err = document.getElementById('error');
    let resDiv = document.getElementById('result');
    err.innerText = "";
    if(!code){ err.innerText="Введи код"; return; }
    if(isNaN(n)){ err.innerText="Введи N"; return; }
    let dec = decode(code);
    if(!dec){ err.innerText="Неверный код"; return; }
    let newId = dec.fullId + BigInt(sign * n);
    if(newId < 0n) newId = 0n;
    let newHi = Number((newId >> 32n) & 0xFFFFFFFFn);
    let newLo = Number(newId & 0xFFFFFFFFn);
    let newCode = encode(newHi, newLo);
    if(!newCode){ err.innerText="Ошибка"; return; }
    resDiv.innerText = newCode;
}

function copy(){
    let txt = document.getElementById('result').innerText;
    if(txt !== "—") {
        navigator.clipboard.writeText(txt);
        alert("Скопировано: " + txt);
    }
}
</script>
</body>
</html>
