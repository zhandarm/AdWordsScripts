# Включение групп по наличию товара на сайте
Скрипт проверяет все активные кампании и остановленные группы. Из группы берет первое объявление и парсит его final url. На странице ищится элемент, который находится между BEGGIN и END. При отсутствии данного элемента группа запускается.

Для скрипта необходимо заполнить двe переменные - BEGGIN и EDN

Настройка селекторов для выбора кампаний, групп, объявлений: https://github.com/promodo2018/AdWordsScripts/blob/master/TUTORIALS/Selectors.md


```js
var BEGGIN = "<div class=\"prd-your-price-numb\">"; //Начальный элемент, от которого начнется парсинг.
                                                    // Некоторые знаки необходимо экранировать, например
                                                    // из " получаем \". Из  \  - \\; 
                                                    
var END = "<span";                                  //Начальный элемент, от которого начнется парсинг

function main(){
  
  var campaign_it = AdWordsApp.campaigns().withCondition("Status = 'ENABLED'")
                              .withCondition("Status != 'REMOVED'").get(); //Селектор кампании
  while(campaign_it.hasNext()){
    var campaign = campaign_it.next();
    var adGroup_it = campaign.adGroups().withCondition("Status = 'DISABLED'").get(); //Селектор группы
    while(adGroup_it.hasNext()){
      var adGroup = adGroup_it.next();
      try{
        var ad = adGroup.ads().get().next();
        var url = ad.urls().getFinalUrl();
        var result = check_url(url);
      }catch(err){
        Logger.log(adGroup.getName() + " " + err);
        continue;
      }
      if(result == null){
        adGroup.enable();
      }
    }
  }
}


function check_url(url){
  
  var params = {muteHttpExceptions:true, followRedirects: false};
  var html_page = UrlFetchApp.fetch(url, params).getContentText();

  var cur = {};
  var ii=0; 
  
  while(ii!=-1){
    ii= html_page.indexOf(BEGGIN,cur);
    if(ii==-1){ break; }
    var priceStart = ii + BEGGIN.length-1;
    cur=priceStart;
    if(priceStart >= 0) {
      var end = html_page.indexOf(END, priceStart);
      var articul = html_page.substr(priceStart, end - priceStart);
    }
  }
  return articul;
}
