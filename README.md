# Noodskii-gold-ai
//@version=6 indicator(“Noodskii Gold AI v2.0.2”, overlay=true,
max_labels_count=500, max_lines_count=500)

//============================== // INPUTS
//============================== grpSess=“Sessions”
session1=input.session(“0730-1100”,“London”,group=grpSess)
session2=input.session(“1200-1500”,“New York”,group=grpSess)
session3=input.session(“2300-0300”,“Asia”,group=grpSess)
tz=input.string(“America/Chicago”,“Timezone”,
options=[“America/New_York”,“America/Chicago”,“America/Denver”,“America/Los_Angeles”,“UTC”],group=grpSess)

grpTrend=“Trend” ema20Len=input.int(20,“EMA20”,group=grpTrend)
ema50Len=input.int(50,“EMA50”,group=grpTrend)
ema200Len=input.int(200,“EMA200”,group=grpTrend)

grpRisk=“Risk” atrLen=input.int(14,“ATR”,group=grpRisk)
slATR=input.float(1.5,“SL ATR Multiplier”,step=.1,group=grpRisk)
tp1ATR=input.float(1.0,“TP1 ATR”,step=.1,group=grpRisk)
tp2ATR=input.float(2.0,“TP2 ATR”,step=.1,group=grpRisk)
tp3ATR=input.float(3.0,“TP3 ATR”,step=.1,group=grpRisk)

//============================== // CORE
//============================== inS1=not
na(time(timeframe.period,session1,tz)) inS2=not
na(time(timeframe.period,session2,tz)) inS3=not
na(time(timeframe.period,session3,tz)) tradeSession=inS1 or inS2 or inS3
sessionName=inS1?“London”:inS2?“New York”:inS3?“Asia”:“Closed”

ema20=ta.ema(close,ema20Len) ema50=ta.ema(close,ema50Len)
ema200=ta.ema(close,ema200Len) vwapVal=ta.vwap(close) atr=ta.atr(atrLen)
volAvg=ta.sma(volume,20)

plot(ema20,color=color.yellow) plot(ema50,color=color.orange)
plot(ema200,color=color.aqua,linewidth=2)
plot(vwapVal,color=color.fuchsia)

bullTrend=close>ema20 and ema20>ema50 and ema50>ema200 and close>vwapVal
bearTrend=close<ema20 and ema20<ema50 and ema50<ema200 and close=90 and
volume>volAvg signalShort=tradeSession and bearScore>=90 and
volume>volAvg

signalTxt=signalLong?“LONG READY”:signalShort?“SHORT READY”:“WAIT”

//============================== // ENTRY / TP / SL
//============================== entry=close longSL=entry-atrslATR
longTP1=entry+atrtp1ATR longTP2=entry+atrtp2ATR longTP3=entry+atrtp3ATR

shortSL=entry+atrslATR shortTP1=entry-atrtp1ATR shortTP2=entry-atrtp2ATR
shortTP3=entry-atrtp3ATR

plot(signalLong?longSL:na,“Long
SL”,color=color.red,style=plot.style_linebr)
plot(signalLong?longTP1:na,“Long
TP1”,color=color.lime,style=plot.style_linebr)
plot(signalLong?longTP2:na,“Long
TP2”,color=color.green,style=plot.style_linebr)
plot(signalLong?longTP3:na,“Long
TP3”,color=color.teal,style=plot.style_linebr)

plot(signalShort?shortSL:na,“Short
SL”,color=color.red,style=plot.style_linebr)
plot(signalShort?shortTP1:na,“Short
TP1”,color=color.lime,style=plot.style_linebr)
plot(signalShort?shortTP2:na,“Short
TP2”,color=color.green,style=plot.style_linebr)
plot(signalShort?shortTP3:na,“Short
TP3”,color=color.teal,style=plot.style_linebr)

plotshape(signalLong,title=“BUY”,location=location.belowbar,color=color.lime,style=shape.triangleup,text=“BUY”)
plotshape(signalShort,title=“SELL”,location=location.abovebar,color=color.red,style=shape.triangledown,text=“SELL”)

alertcondition(signalLong,“Long Ready”,“Noodskii Gold AI Long Ready”)
alertcondition(signalShort,“Short Ready”,“Noodskii Gold AI Short Ready”)

//============================== // DASHBOARD
//============================== var table
d=table.new(position.top_right,2,7,border_width=1) if barstate.islast
table.cell(d,0,0,“Session”) table.cell(d,1,0,sessionName)
table.cell(d,0,1,bullTrend?“Bull Score”:“Bear Score”)
table.cell(d,1,1,str.tostring(bullTrend?bullScore:bearScore)+“%”)
table.cell(d,0,2,“Trend”)
table.cell(d,1,2,bullTrend?“Bullish”:bearTrend?“Bearish”:“Neutral”)
table.cell(d,0,3,“VWAP”) table.cell(d,1,3,close>vwapVal?“Above”:“Below”)
table.cell(d,0,4,“Volume”)
table.cell(d,1,4,volume>volAvg?“High”:“Normal”) table.cell(d,0,5,“ATR”)
table.cell(d,1,5,atr>ta.sma(atr,20)?“High”:“Normal”)
table.cell(d,0,6,“Signal”) table.cell(d,1,6,signalTxt)
