OwnJS/抖音.js /
@67here
67here Create 抖音.js
Latest commit 6c83187 on 27 Oct 2021
 History
 1 contributor
401 lines (359 sloc)  16.8 KB

'ui';

ui.layout(
    <scroll>
    <vertical padding="16">
        <text textSize="16sp" textColor="blue" text="Tips:"/>
        <text textSize="16sp" textColor="blue" text="1）适配抖音版本：17.2.0"/>
        <text textSize="16sp" textColor="blue" text="2）按下音量+，结束脚本"/>
        <text textSize="16sp" textColor="blue" text="3）用xx1，xx2...xx5，指代变量"/>

        <button id='system_setting' text="开启无障碍" textSize="20sp"/>

        <text textSize="16sp" textColor="red" text="1.批量抖音号(要求准确无误)"/>
        <input id="tiktok" hint="换行---分割线"/>

        <text textSize="16sp" textColor="red" text="2.随机评论(视频)"/>
        <input id="comment_video" hint="主句"/>
        <input id="random_video" hint="变量库--换行隔开"/>

        <text textSize="16sp" textColor="red" text="3.随机评论(私信)"/>
        <input id="comment_message" hint="主句"/>
        <input id="random_message" hint="变量库--换行隔开"/>

        <text textSize="16sp" textColor="red" text="4.随机间隔时间(最长)"/>
        <input id="time_most" inputType="number" hint="输入'5',0-5s内的随机一个时间"/>

        <horizontal>
            <text textSize="16sp" textColor="red" text="5.粉丝数高于"/>
            <input id="num_fans" hint=""/>
            <text textSize="16sp" textColor="red" text="时，跳过"/>
        </horizontal>

        <text textSize="20sp" textColor="black" text="------------------------------------------"/>
        <text textSize="20sp" textColor="blue" text="【规则】关注+点赞+私信"/>
        <text textSize="20sp" textColor="black" text="------------------------------------------"/>

        <text textSize="16sp" textColor="red" text="一、轮流执行"/>
        <horizontal>
            <input id="time_video" inputType="number"/>
            <text textSize="16sp" textColor="black" text="次"/>
            <text textSize="16sp" textColor="black" text="视频留言;"/>
            <input id="time_meg" inputType="number"/>
            <text textSize="16sp" textColor="black" text="次"/>
            <text textSize="16sp" textColor="black" text="私信，自动关注"/>
        </horizontal>
        <text textSize="16sp" textColor="red" text="二、由作品日期决定"/>
        <horizontal>
            <text textSize="16sp" textColor="black" text="1）最新作品发布于"/>
            <input id="num_date1" inputType="number"/>
            <text textSize="16sp" textColor="black" text="天内，关注"/>
        </horizontal>
        <horizontal>
            <text textSize="16sp" textColor="black" text="2）最新作品发布于"/>
            <input id="num_date2" inputType="number"/>
            <text textSize="16sp" textColor="black" text="天内，点赞+评论"/>
        </horizontal>
        <text textSize="16sp" textColor="black" text="3）无作品/私密账号/搜索不到，跳过"/>

        <horizontal>
        <button id='start1' text="运行[规则一]" textSize="16sp"/>
        <button id='start2' text="运行[规则二]" textSize="16sp"/>
        <button id='save' text="保存配置" textSize="16sp"/>
        </horizontal>

        <horizontal>
        <button id='ck_video' text="测试评论(视频)" textSize="20sp"/>
        <button id='ck_message' text="测试评论(私信)" textSize="20sp"/>
        </horizontal>
        <text textSize="16sp" textColor="red" text="测试结果"/>
        <text id="output"></text>
    </vertical>
    </scroll>
)

ui.system_setting.click(function(){
    auto();
    if (floaty && floaty.hasOwnProperty("checkPermission") && !floaty.checkPermission()) {
        floaty.requestPermission(); toast("请先开启悬浮窗权限再运行,否则无法显示提示"); exit()
    }
})
ui.save.click(function(){
    let ids = ui.tiktok.text();
    let comments_video = ui.comment_video.text();
    let random_video = ui.random_video.text();
    let comment_message = ui.comment_message.text();
    let random_message = ui.random_message.text();
    let times = ui.time_most.text();
    let num_fans = ui.num_fans.text();

    let time_video = ui.time_video.text();
    let time_meg = ui.time_meg.text();

    let num_date1 = ui.num_date1.text();
    let num_date2 = ui.num_date2.text();

    storage.put('save_id', ids);
    storage.put('save_video', comments_video);
    storage.put('random_video', random_video);
    storage.put('save_message', comment_message);
    storage.put('random_message', random_message);
    storage.put('save_time', times);
    storage.put('num_fans', num_fans);

    storage.put('time_video', time_video);
    storage.put('time_meg', time_meg);

    storage.put('num_date1', num_date1);
    storage.put('num_date2', num_date2);
    toast('配置已保存');
})

ui.start1.click(function(){
    var id_array = ui.tiktok.text().split('\n');
    var video_array = ui.comment_video.text().split('\n');
    var video_random_array = ui.random_video.text().split('\n');
    var message_array = ui.comment_message.text().split('\n');
    var message_random_array = ui.random_message.text().split('\n');
    var times = ui.time_most.text();
    var num_fans = ui.num_fans.text();

    var time_video = ui.time_video.text();
    var time_meg = ui.time_meg.text();
    var t_video = time_video;
    var t_meg = time_meg;
    log('输入抖音号'+id_array.length+'个\n执行规则一');


    threads.start(function(){
        console.show();
        sleep(1000);
          for(i = 0; i < id_array.length; i++){
            log('\n开始执行第'+(i+1)+'个');

            if(t_video) {
                t_video--;
                log('本次视频留言');
                a_loop(id_array[i], video_array, video_random_array, message_array, message_random_array, num_fans, 0);
            }
            else if(t_meg) {
                t_meg--;
                log('本次私信');
                a_loop(id_array[i], 0, 0, message_array, message_random_array, num_fans, 0);
            }
            else {
                t_video = time_video;
                t_meg = time_meg;
                i = i - 1;
                continue;
            }
            let now_time = random(0, times);
            toastLog('等待'+now_time+'秒');
            sleep(now_time*1000);
            let id_rray = storage.get("save_id");
            let id_ray = id_rray.split('\n');
            id_ray = id_ray.splice(1);
            log('存储更新,当前剩余抖音号:'+id_ray.length);
            id_rray = id_ray.join('\n');
            storage.put('save_id', id_rray);
        };
        toastLog('任务完成，脚本结束');
        console.hide();
        exit();
    })
})

ui.start2.click(function(){
    var id_array = ui.tiktok.text().split('\n');
    var video_array = ui.comment_video.text().split('\n');
    var video_random_array = ui.random_video.text().split('\n');
    var message_array = ui.comment_message.text().split('\n');
    var message_random_array = ui.random_message.text().split('\n');
    var times = ui.time_most.text();
    var num_fans = ui.num_fans.text();

    var num_date1 = ui.num_date1.text();
    var num_date2 = ui.num_date2.text();  

    log('输入抖音号'+id_array.length+'个\n执行规则二');
    threads.start(function(){
        console.show();
        sleep(1000);
          for(i = 0; i < id_array.length; i++){
            log('\n开始执行第'+(i+1)+'个');
            a_loop(id_array[i], video_array, video_random_array, message_array, message_random_array, num_fans, 1, num_date1, num_date2);

            let now_time = random(0, times);
            toastLog('等待'+now_time+'秒');
            sleep(now_time*1000);
            let id_rray = storage.get("save_id");
            let id_ray = id_rray.split('\n');
            id_ray = id_ray.splice(1);
            log('存储更新,当前剩余抖音号:'+id_ray.length);
            id_rray = id_ray.join('\n');
            storage.put('save_id', id_rray);
        };
        toastLog('任务完成，脚本结束');
        console.hide();
        exit();
    })
})
//测试视频
ui.ck_video.click(function(){
    var video_array = ui.comment_video.text().split('\n');
    var video_random_array = ui.random_video.text().split('\n');
    ui.output.setText(random_words(video_array, video_random_array));
})
//测试私信
ui.ck_message.click(function(){
    var message_array = ui.comment_message.text().split('\n');
    var message_random_array = ui.random_message.text().split('\n');
    ui.output.setText(random_words(message_array, message_random_array));
})
//读取配置
var storage = storages.create("67here12345");
if(storage.get("save_id")) ui.tiktok.setText(storage.get("save_id"));
if(storage.get("save_video")) ui.comment_video.setText(storage.get("save_video"));
if(storage.get("random_video")) ui.random_video.setText(storage.get("random_video"));
if(storage.get("save_message")) ui.comment_message.setText(storage.get("save_message"));
if(storage.get("random_message")) ui.random_message.setText(storage.get("random_message"));
if(storage.get("save_time")) ui.time_most.setText(storage.get("save_time"));
if(storage.get("num_fans")) ui.num_fans.setText(storage.get("num_fans"));

if(storage.get("time_video")) ui.time_video.setText(storage.get("time_video"));
if(storage.get("time_meg")) ui.time_meg.setText(storage.get("time_meg"));

if(storage.get("num_date1")) ui.num_date1.setText(storage.get("num_date1"));
if(storage.get("num_date2")) ui.num_date2.setText(storage.get("num_date2"));

//点击不可点击的控件
function position_click(x){
    if(x) click(x.bounds().centerX(), x.bounds().centerY())
        else toastLog('找不到此控件');}
//子线程 音量键关闭
threads.start(function(){
    events.observeKey();
    events.onKeyDown("volume_up", function(){toastLog("音量+被按下,结束脚本！");sleep(1000);console.hide();exit();});
})
//0.一次循环
function a_loop(id2, v, v_array, o, o_array, num_fans, select_mode, mode2_days1, mode2_days2){
    if(getInto(id2, num_fans) == 1) return false;

    sleep(1000);
    if(text('作品 0').findOne(3000)) {
        if(select_mode){log('无作品，直接跳过');}
        else{log('当前无作品\n只进行私信操作');round_only_message(o, o_array);};
    }
    else{
        log('进入作品中...');
        sleep(1000);
        className('android.view.View').descContains("点赞数").findOne().click();  
        className("android.widget.LinearLayout").descContains("喜欢").waitFor();
        log('已进入作品');sleep(1000);
        if(!select_mode){
            className("android.widget.LinearLayout").descContains("喜欢").findOne().click();log('点赞成功');sleep(2000);
            if(!v) {back();round_only_message(o, o_array);}
            else{
                text("留下你的精彩评论吧").findOne().setText(random_words(v, v_array));sleep(1000);
                desc('发送').findOne().click();toastLog('已评论');sleep(1000);
            }
            back();sleep(2000);

            if(id('hp7').text('关注').findOne(3000)) {id('hp7').text('关注').findOne(2000).parent().click();toastLog('已关注');}
            else toastLog('早已关注');
            if(text('知道了').findOne(2000)) text('知道了').findOne(2000).click();
            sleep(2000);
        }
        else{
            let value1 = date_compare(mode2_days1);
            let value2 = date_compare(mode2_days2);
            if(value2) {
                log('时间内! 评论+点赞');
                sleep(1000);
                className("android.widget.LinearLayout").descContains("喜欢").findOne().click();log('点赞成功');sleep(2000);
                text("留下你的精彩评论吧").findOne().setText(random_words(v, v_array));sleep(1000);
                desc('发送').findOne().click();toastLog('已评论');sleep(1000);
            }
            else {log('时间外，不评');}

            while(!text('关注').findOne(3000)) back();

            if(value1) {
                log('时间内! 关注！');
                if(id('hp7').text('关注').findOne(3000)) {id('hp7').text('关注').findOne(2000).parent().click();toastLog('已关注');}
                else toastLog('早已关注');
                if(text('知道了').findOne(2000)) text('知道了').findOne(2000).click();
                sleep(2000);
            }
            else log('时间外 不关注');
        }
    }
    comeback();
}
//1.回到首页
function comeback(){
    while(!className('android.widget.Button').desc('搜索').findOne(2000)) back();
    log('回到首页');
}
//2.随机话
function random_words(words_array, words_random_array){
    let length1 = words_array.length-1;
    let length2 = words_random_array.length-1;
    let words = words_array[random(0, length1)]

    if(words.indexOf('xx1') != -1) words = words.replace('xx1', words_random_array[random(0, length2)]);
    if(words.indexOf('xx2') != -1) words = words.replace('xx2', words_random_array[random(0, length2)]);
    if(words.indexOf('xx3') != -1) words = words.replace('xx3', words_random_array[random(0, length2)]);
    if(words.indexOf('xx4') != -1) words = words.replace('xx4', words_random_array[random(0, length2)]);
    if(words.indexOf('xx5') != -1) words = words.replace('xx5', words_random_array[random(0, length2)]);
    return words;
}

//1.启动抖音、进入个人首页
function getInto(tik_id, num_fans){
        var abc = 1;
        sleep(1000);home();sleep(1000);
        app.launchPackage('com.ss.android.ugc.aweme');sleep(1000);comeback();
        className('android.widget.Button').desc('搜索').waitFor();sleep(1000);
        className('android.widget.Button').desc('搜索').findOne().parent().parent().click();sleep(500);
        textMatches('搜索|取消').waitFor();sleep(1000);
        className('android.widget.EditText').editable(true).setText(tik_id+'@');
        let search = className('android.widget.TextView').text('搜索').findOne();
        position_click(search);
        sleep(2000);text('用户').findOne().parent().click();sleep(2000);
        for(i2 = 0; i2 < 1; i2++){
            let judge_wait1 = descContains(tik_id).findOne(10000);
            sleep(2000);
            if(!judge_wait1) {log('未搜索到结果，直接跳过...');return abc;};
            let origin_text = judge_wait1.desc().slice(judge_wait1.desc().indexOf('粉丝'), judge_wait1.desc().indexOf('抖音号'));
            let nums = origin_text.match(/\d+(\.\d+)?/g);
            if(origin_text.indexOf("w") != -1) nums = nums * 10000;
            log('粉丝数为:'+nums);
            if(Number(nums) > num_fans) {log('粉丝数高于设定，跳过');return abc;}
            sleep(1000);
            console.hide();
            sleep(1000);
            while(!text('获赞').findOne(3000)) position_click(judge_wait1);
            sleep(1000);
            console.show();
        }
    sleep(2000);
    if(text('求更新').findOne(2000)) text('求更新').findOne(2000).parent().parent().parent().click();
    if(text('私密账号').findOne(2000)) {log('私密账号，跳过'); return abc;}
}


// 只留言
function round_only_message(o, o_array){
    log("开始私信");
    desc('更多').findOne().click();sleep(1000);
    text('发私信').findOne().parent().click();
    text('发送消息…').waitFor();
    text('发送消息…').findOne().setText(random_words(o, o_array));
    desc('发送').waitFor();sleep(1000);
    desc('发送').findOne().click();
    toastLog('已私信');        
}

//日期比较1
function date_compare(days){
    let target = id('k-+').findOne().text();
    let result_array,result;
    let actuall_date = new Array();
    curTime = new Date();

    if(target.indexOf('周前') != -1) result = 7
    else if(target.indexOf('天前') != -1) {result_array = target.match(/\d+(\.\d+)?/g);result = result_array[0];}
    else if(target.indexOf('月') != -1) {
        result_array = target.match(/\d+(\.\d+)?/g);
        let image_results = days_before(days);

        actuall_date[1] = Number(result_array[0]);
        actuall_date[2] = Number(result_array[1]);
        if(result_array.length == 3) actuall_date[0] = Number(result_array[0])
        else actuall_date[0] = curTime.getFullYear();

        if(actuall_date[0] > image_results[0]) return true
        else if(actuall_date[0] < image_results[0]) return false
        else if(actuall_date[1] > image_results[1]) return true
        else if(actuall_date[1] < image_results[1]) return false
        else if(actuall_date[2] > image_results[2]) return true
        else return false
    }
    else result = 0.5;
    if(Number(result) > Number(days)) return false
    else return true;
}
//日期比较2
function days_before(days){
    var  startDate  =  new  Date  ();
    var  intValue  =  0; 
    var  endDate  =  null; 

    intValue  =  startDate.getTime();
    intValue  =  intValue - days  *  (24  *  3600  *  1000); 
    endDate  =  new  Date  (intValue); 
    let result_1 = new Array();
    result_1[0] = endDate.getFullYear();
    result_1[1] = endDate.getMonth() + 1;
    result_1[2] = endDate.getDate();
    return result_1;
}
Footer
© 2022 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
