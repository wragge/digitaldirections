---
layout: slide
title: ""
---

{% highlight javascript %}
// ==UserScript==
// @name           RecordSearch -- Redactions
// @namespace      http://timsherratt.org/recordsearch-redactions
// @description    Access is not always accessible. This userscript enriches the National Archives of Australia's database by inserting information about redactions in ASIO files.
// @version        0.1
// @date           2016-10-15
// @creator        Tim Sherratt
// @include        http://recordsearch.naa.gov.au/SearchNRetrieve/Interface/ListingReports/ItemsListing.aspx*
// @include        http://recordsearch.naa.gov.au/SearchNRetrieve/Interface/DetailsReports/ItemDetail.aspx*
// @grant          GM_xmlhttpRequest
// @connect        owebrowse.herokuapp.com
// ==/UserScript==
var processed_series = ['A6119'];

function getLink() {
    if (links.length > 0) {
        if (type == 'single') {
            var total = 1;
        } else {
            var total = links.length;
        }
        if (link < total ) {
            //getPages(rs_links.snapshotItem(link));
            getPages(links[link]);
        }
    }
}
function getPages(rs_link) {
    var barcode = rs_link.match(/\/ViewImage.aspx\?B=(\d+)/)[1];
    var url = 'https://owebrowse.herokuapp.com/items/' + barcode + '/redactions/?format=json';
    GM_xmlhttpRequest({
        method: 'GET',
        url: url,
        onload: function(response) {
            var data = JSON.parse(response.responseText);
            if (type == 'single') {
                if (data.total) {
                    var table = document.evaluate('//*[@id="ctl00_ContentPlaceHolderSNR_ucItemDetails_ctl01"]/tbody/tr/td/div/table/tbody', document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
                    var cell = document.evaluate('//*[text()="Reason for restriction"]', document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
                    var row = cell.parentNode.parentNode;
                    var newRow = document.createElement('tr');
                    var titleCell = document.createElement('td');
                    var titleDiv = document.createElement('div');
                    var att = document.createAttribute("class");
                    att.value = 'field';
                    titleDiv.setAttributeNode(att);
                    titleCell.setAttributeNode(att.cloneNode());
                    var contentCell = document.createElement('td');
                    var rowTitle = document.createTextNode("Redactions");
                    titleDiv.appendChild(rowTitle);
                    titleCell.appendChild(titleDiv);
                    var block = document.createElement('div');
                    var att = document.createAttribute("style");
                    percent = 100 - (Math.log10((100 * data.percentage)) * 25);
                    color = 'rgb(' + percent + '%, ' + percent + '%, ' + percent + '%)';
                    att.value = 'background: ' + color + '; width: 100px; height: 20px; border: 1px solid black';
                    block.setAttributeNode(att);
                    contentCell.appendChild(block);
                    contentCell.innerHTML = contentCell.innerHTML + '<br>Number of redactions: ' + data.total + '<br>Area redacted: ' + data.percentage.toFixed(2) + '%';
                    newRow.appendChild(titleCell);
                    newRow.appendChild(contentCell);
                    table.insertBefore(newRow, row);
                }
            } else {
                if (data.total) {
                    xpath = "//a[contains(@href, '" + barcode + "')]/img";
                    img = document.evaluate(xpath, document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
                    var rs_link = img.parentNode;
                    var cell = img.parentNode.parentNode;
                    var block = document.createElement('div');
                    var att = document.createAttribute("style");
                    percent = 100 - (Math.log10((100 * data.percentage)) * 25);
                    color = 'rgb(' + percent + '%, ' + percent + '%, ' + percent + '%)';
                    att.value = 'background: ' + color + '; width: 60px; height: 20px; border: 1px solid black';
                    block.setAttributeNode(att);
                    rs_link.replaceChild(block, img);
                    cell.innerHTML = cell.innerHTML + '<br>Number of redactions: ' + data.total + '<br>Area redacted: ' + data.percentage.toFixed(2) + '%';
                }
            }
            link++;
            getLink();
        }
    });
}
var links = [];
if (document.location.href.indexOf('ItemDetail.aspx') > 0) {
    var type = 'single';
    series = document.evaluate("//a[contains(@href, '/SeriesDetail.aspx')]", document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue.textContent;
    digitised = document.evaluate("//a[contains(@href, '/ViewImage.aspx')]", document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
    if (digitised && processed_series.indexOf(series) != -1) {
        links.push(digitised.href);
    }
} else {
    var type = 'multiple';
    var rows = document.evaluate('//*[@id="ctl00_ContentPlaceHolderSNR_tblItemDetails"]/tbody/tr', document, null, XPathResult.ORDERED_NODE_SNAPSHOT_TYPE, null);
    for ( var i=0 ; i < rows.snapshotLength; i++ ) {
        var series = document.evaluate('td[2]', rows.snapshotItem(i), null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue.textContent;
        var digitised = document.evaluate('td[6]/a', rows.snapshotItem(i), null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
        if (digitised && processed_series.indexOf(series) != -1) {
            links.push(digitised.href);
        }
    }
}
var link = 0;
getLink(link);
{% endhighlight %}

###### [https://gist.github.com/wragge/b6d120a712ae7d598ab70e228f070bac](https://gist.github.com/wragge/b6d120a712ae7d598ab70e228f070bac){: .external}
