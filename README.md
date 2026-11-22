# BiSaiChouQian
乒乓球分组抽签工具规则：① 同公司选手必须分到不同组；② 同公司人数尽量平均分到上半区(A–D)与下半区(E–H)；③ 已确定的种子选手固定在指定组，只补该组其余名额；④ 每次随机可重抽。所有计算都在本机浏览器完成，不上传任何数据。
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>乒乓球抽签｜8组A-H 上下半区+同公司回避+均分半区</title>
<style>
*{box-sizing:border-box;font-family:system-ui,sans-serif;}
body{padding:20px;background:#f4f7fa;}
.box{background:#fff;padding:16px;border-radius:10px;margin-bottom:12px;box-shadow:0 2px 8px #00000015;}
textarea{width:100%;height:140px;padding:8px;font-size:14px;}
button{padding:8px 16px;margin:4px;background:#2563eb;color:#fff;border:none;border-radius:6px;cursor:pointer;font-size:14px;}
#result{white-space:pre-wrap;font-size:14px;line-height:1.6;}
.warn{color:#dc2626;font-weight:bold;}
.info{color:#16a34a;}
</style>
</head>
<body>
<div class="box">
<h3>📋 选手名单（格式：姓名,公司 一行一个）</h3>
<textarea id="playerList" placeholder="示例：
张三,中原石油
李四,中原石油
王五,XX公司"></textarea>
</div>
<div class="box">
<h3>🌱 种子选手固定（格式：姓名=组号，组号A~H，一行一个）</h3>
<textarea id="seedList" placeholder="示例：
张三=A
李四=E"></textarea>
</div>
<div class="box">
<h3>⚙️ 分组规则：A/B/C/D=上半区｜E/F/G/H=下半区，一共8组</h3>
<p>约束：同公司不同组；同公司人员尽量均分在上/下半区</p>
<button onclick="doDraw()">开始抽签</button>
<button onclick="copyResult()">复制结果</button>
<button onclick="resetAll()">清空</button>
</div>
<div class="box">
<h3>📄 抽签结果</h3>
<div id="result"></div>
</div>

<script>
const GROUPS = ["A","B","C","D","E","F","G","H"];
const TOP = new Set(["A","B","C","D"]); //上半区
const BOT = new Set(["E","F","G","H"]); //下半区

function parseText(txt){
    const arr = [];
    txt.trim().split("\n").forEach(line=>{
        const l = line.trim();
        if(!l) return;
        const sp = l.split(",");
        if(sp.length>=2) arr.push({name:sp[0].trim(),corp:sp[1].trim()});
    })
    return arr;
}
function parseSeed(txt){
    const seedMap = {};
    txt.trim().split("\n").forEach(line=>{
        const l = line.trim();
        if(!l) return;
        const sp = l.split("=");
        if(sp.length>=2){
            const name = sp[0].trim();
            const g = sp[1].trim().toUpperCase();
            if(GROUPS.includes(g)) seedMap[name] = g;
        }
    })
    return seedMap;
}

function doDraw(){
    const players = parseText(document.getElementById("playerList").value);
    const seedMap = parseSeed(document.getElementById("seedList").value);
    const seedNames = new Set(Object.keys(seedMap));
    //校验种子是否在选手列表
    for(const sn of seedNames){
        if(!players.find(p=>p.name===sn)){
            document.getElementById("result").innerHTML = `<span class='warn'>错误：种子【${sn}】不在选手名单内！</span>`;
            return;
        }
    }
    //初始化分组
    const groupData = {};
    GROUPS.forEach(g=>groupData[g]=[]);
    //先放种子
    for(const [name,g] of Object.entries(seedMap)){
        const p = players.find(x=>x.name===name);
        groupData[g].push(p);
    }
    //剩余选手（排除种子）
    const restPlayers = players.filter(p=>!seedNames.has(p.name));
    //多次随机尝试，找满足约束的方案
    let finalAns = null;
    const MAXTRY = 12000;
    for(let t=0;t<MAXTRY;t++){
        const gClone = JSON.parse(JSON.stringify(groupData));
        const shuffle = [...restPlayers].sort(()=>Math.random()-0.5);
        let ok = true;
        for(const p of shuffle){
            const corp = p.corp;
            //已经有该公司的组，不能再进
            const usedGroup = new Set();
            for(const g of GROUPS){
                for(const m of gClone[g]){
                    if(m.corp === corp) usedGroup.add(g);
                }
            }
            //可选组：不在usedGroup里的组
            const canGroup = GROUPS.filter(g=>!usedGroup.has(g));
            if(canGroup.length===0){ok=false;break;}
            //优先选择：让该公司上下半区数量更均衡
            let bestG = null;
            let bestScore = 9999;
            for(const g of canGroup){
                //统计该公司当前在上、下半区数量
                let cntTop=0,cntBot=0;
                for(const gg of GROUPS){
                    for(const m of gClone[gg]){
                        if(m.corp!==corp) continue;
                        if(TOP.has(gg)) cntTop++;
                        else cntBot++;
                    }
                }
                let newTop = cntTop + (TOP.has(g)?1:0);
                let newBot = cntBot + (BOT.has(g)?1:0);
                const diff = Math.abs(newTop - newBot);
                if(diff < bestScore){
                    bestScore = diff;
                    bestG = g;
                }
            }
            gClone[bestG].push(p);
        }
        if(ok){
            finalAns = gClone;
            break;
        }
    }
    if(!finalAns){
        document.getElementById("result").innerHTML = `<span class='warn'>尝试${MAXTRY}次，无法满足全部约束！<br>排查：某公司人数超过8人 / 种子固定后剩余位置冲突</span>`;
        return;
    }
    //输出文本
    let out = "===== 乒乓球抽签结果（A-D上半区，E-H下半区）=====\n";
    for(const g of GROUPS){
        out += `【${g}组】${TOP.has(g)?"(上半区)":"(下半区)"}\n`;
        finalAns[g].forEach((it,idx)=>{
            const mark = seedMap[it.name]===g?"【种子】":"";
            out += `  ${idx+1}. ${it.name}｜${it.corp} ${mark}\n`;
        })
        out += "\n";
    }
    document.getElementById("result").innerText = out;
}
function copyResult(){
    const txt = document.getElementById("result").innerText;
    navigator.clipboard.writeText(txt).then(()=>alert("已复制！"));
}
function resetAll(){
    document.getElementById("playerList").value="";
    document.getElementById("seedList").value="";
    document.getElementById("result").innerText="";
}
</script>
</body>
</html>
