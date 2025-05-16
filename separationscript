
function main() {
    var campaignId = '22437518598'; 
    var brandedKeywords = ['theragun', 'jetboots', 'jet boots', 'thera face', 'theraface', 'therabody', 'theragun', 'theraface', 'thera']; 
    var negativeKeywordListNames = ['Therabody NKW Script']; 

    negativeKeywordListNames.forEach(function(listName) {
        removeAllNegativeKeywordsFromList(listName);
    });

    var keywordsToAdd = collectKeywordsToExclude(campaignId, brandedKeywords);
    distributeKeywordsAcrossLists(keywordsToAdd, negativeKeywordListNames);
}

function removeAllNegativeKeywordsFromList(name) {
    const negativeKeywordLists = AdsApp.negativeKeywordLists()
        .withCondition(`Name = "${name}"`)
        .get();
    if (!negativeKeywordLists.hasNext()) {
        throw new Error(`Cannot find negative keyword list with name "${name}"`);
    }
    const negativeKeywordList = negativeKeywordLists.next();

    var negativeKeywordsIterator = negativeKeywordList.negativeKeywords().get();
    while (negativeKeywordsIterator.hasNext()) {
        var negativeKeyword = negativeKeywordsIterator.next();
        negativeKeyword.remove();
    }
    Logger.log('Removed all negative keywords from list: ' + name);
}

function collectKeywordsToExclude(campaignId, brandedKeywords) {
    var keywordsToAdd = [];
    var report = AdsApp.report(
        "SELECT Query " +
        "FROM SEARCH_QUERY_PERFORMANCE_REPORT " +
        "WHERE CampaignId = '" + campaignId + "' " +
        "AND CampaignStatus = ENABLED " +
        "AND AdGroupStatus = ENABLED"
    );

    var rows = report.rows();
    while (rows.hasNext()) {
        var row = rows.next();
        if (!doesPhraseMatchAny(row['Query'], brandedKeywords)) {
            keywordsToAdd.push('[' + row['Query'] + ']'); 
        }
    }
    return keywordsToAdd;
}

function doesPhraseMatchAny(searchTerm, brandedKeywords) {
    return brandedKeywords.some(brandedKeyword => searchTerm.toLowerCase().includes(brandedKeyword.toLowerCase()));
}

function distributeKeywordsAcrossLists(keywords, listNames) {
    var MAX_KEYWORDS_PER_LIST = 5000;
    var remainingKeywords = keywords.slice(); 

    for (var i = 0; i < listNames.length && remainingKeywords.length > 0; i++) {
        var list = getOrCreateNegativeKeywordList(listNames[i]);
        var keywordsToAdd = remainingKeywords.slice(0, MAX_KEYWORDS_PER_LIST);
        
        try {
            list.addNegativeKeywords(keywordsToAdd);
            Logger.log('Added ' + keywordsToAdd.length + ' keywords to ' + listNames[i]);
            remainingKeywords = remainingKeywords.slice(MAX_KEYWORDS_PER_LIST); 
        } catch (e) {
            Logger.log('Attempt to add to ' + listNames[i] + ' failed: ' + e.message);
            // Potentially full list or other error; try next list without slicing remainingKeywords
        }
        
        if (remainingKeywords.length === 0) {
            Logger.log('All keywords have been successfully distributed across the lists.');
            break;
        }
    }
    
    if (remainingKeywords.length > 0) {
        Logger.log('Not all keywords were added. Consider adding more lists or increasing the limit.');
    }
}

function getOrCreateNegativeKeywordList(listName) {
    var lists = AdsApp.negativeKeywordLists().withCondition('Name = "' + listName + '"').get();
    if (lists.hasNext()) {
        return lists.next();
    } else {
        return AdsApp.newNegativeKeywordListBuilder().withName(listName).build().getResult();
    }
}
