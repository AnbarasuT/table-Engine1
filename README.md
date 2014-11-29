actvity.js
============


var mainParent, interactiveContainer, pageContainer, page0, activityHolder,ScrollPercent;
var feedback;
var blocker;
var eventType = "click";
var activityName = "StatisticsSortingTable";
var activityVersion = "1.0";
var instructionDivs;
var scrollPos = 0;

var statisticsToDisplay = ["Sum", "Mean", "Median", "Mode", "Range", "Interquatile range", "First quartile Q1", "Third quartile Q3", "Standard deviation", "Mean absolute deviation"];
var isFirstPopupOpen
var container_layout,alignContan_Btn, container_table, container_options, container_popups;
var manifest_copy_str;
var warningPopup;
var popupDraggable;
var scrollbarDraggable;
var simpleKeypad, isFirstKeyPadOpen=false;

$(document).ready(function() {
    if ("ontouchstart" in document.documentElement) eventType = "touchstart";
    else eventType = "mousedown";
    
    mainParent = $("body");
    mainParent.bind("touchmove", function(e){ e.preventDefault(); });
    interactiveContainer = $("<div id=\"interactive-container\"></div>");
    pageContainer = $("<div id=\"page-container\"></div>");
    page0 = $("<div id=\"page0\" class=\"page\"></div>");
    instructionDivs=$("<div id=\"blocker\" class=\"blocker\" onclick=\"closeInstruction()\"></div><div class=\"IOContainer\" onclick=\"closeInstruction()\"><div class=\"IOHolder\"><div class=\"IOBox\"><div style=\"position:absolute; left:2%; top:5%; opacity:0.6;\" id=\"instructions-button\"><img src=\"images/btInfoOn.png\" width=\"25px\" height=\"27px\" /></div><span id=\"IOContent\" style=\"text-align:left; font-family:myFontfamily; font-size:21px;left:40px; line-height:1.4em; width:390px; position:absolute; top:45px;\">"+
    
    (manifest.instruction ? manifest.instruction : "")+
    
    "</span><div class=\"closeBtn\" >&#x000D7;</div><div style=\"position:absolute; left:43%; top:75%;\" ></div></div></div></div>");
    
    activityHolder = $("<div id=\"activityHolder\"></div>");
    blocker = $("<div id=\"generalBlocker\" class=\"blocker\" style=\"z-index:25;\" ></div>");
    feedback = $("<div class=contentFeedback> <div class=btReset><div class=icoReset></div> </div> <div class=\"btInfo\"><div class=\"icoInfo\" ></div></div><div class=\"btFeedback\"> <div class=\"icoFeed\"></div> </div> <div class=\"score\"> <div id=\"contCorrect\">0 <img src=\"images/IconCorrect.png\"></div><div id=\"contIncorrect\">0 <img src=\"images/IconIncorrect.png\"></div> </div>  </div> ");
    warningPopup = $("<div class=\"warn_container\"><div class=\"warn_holder\"><div class=\"warn_box\"><div class=\"warn_content\"></div><div class=\"btn_warn_ok\">OK</div></div></div></div>");
    
    page0.append(activityHolder).append(feedback).append(blocker).append(warningPopup);
    pageContainer.append(page0);
    interactiveContainer.append(pageContainer);
    mainParent.append(interactiveContainer);
    interactiveContainer.append(instructionDivs);
    
    blocker.hide();
    
    $(".icoReset").bind(eventType, resetClick);
    $(".icoInfo").bind("click",showInstruction);
    $(".btn_warn_ok").bind(eventType, hideWarningPopup);
    
    
    
    
    /*Activity Starts*/

    setResetState(false);
    showInstruction();
    
    if (!window.manifest) {
	$("#activityHolder").text("Manifest is missing.");
	return;
    }
    
    if (!manifest.options) manifest.options = {};
    
    manifest_copy_str=JSON.stringify(window.manifest);
    
    activityName = manifest.activityName;
    
    formBasicTemplate();
    formActivity();
    formOptions();
    closePopup();
    
    simpleKeypad = new vKeyPad(".datasetTxtBox", "#activityHolder", {keys:[1,2,3,4,5,6,7,8,9,0], maximumLength:3});
    var keypadDrag = new Drag(simpleKeypad.KeyPadHolder);
    simpleKeypad.isAutoAdjustPosition = false;
    simpleKeypad.onTextBoxFocus = function(){
	$('#chk_statistics_dataset_0').prop('checked',false);
	$('#chk_statistics_dataset_1').prop('checked',true);
	if(this.isCoveringTextBoxes() || !isFirstKeyPadOpen) {
	    var left = container_popups.position().left;
	    var top = container_popups.position().top;
	    var width = simpleKeypad.KeyPadHolder.width();
	    var width2 = container_popups.width();
	    if (left > width)
		left -= width;
	    else left += width2;
	    this.KeyPadHolder.css({"left":left+"px", "top":top+"px"});
	}
	isFirstKeyPadOpen = true;
    }
    
    window.onresize = alignPositions;
    
    /*Activity Ends*/
    
    setTimeout(function(){
	$("body").css({"-webkit-transform":"scale(1)"});
	$(".icoReset").css({"opacity":"0.5"});
	eventBroker = _({}).extend(require("chaplin/lib/event_broker"));
	eventBroker.publishEvent("#fetch", { type : "state" }, function(state) {
	    if (state) {
		var savedState = JSON.parse(state);
		if(savedState["activityName"] == activityName &&  savedState["activityVersion"] == activityVersion)
		    _.each(savedState, function(value, key, list) {
			if (key == "manifest") window.manifest = value;
			else if (key=="ScrollPercent") ScrollPercent=value;
			else if (key == "div_scrollButtonPos") {
			    scrollPos = value;
			    $("#div_scrollButton").css({"top":value+"px"});
			    setScrollerProp($("#div_scrollButton"), true);
			    
			    //$('#headerTable tr>th:first-child').css({'border-right':'0px solid #9ECE61'});
			    //$('#div_tableHeader').css({'border-right':'0px solid #000'});
			}else if (key=="icoReset") $('.icoReset').css('opacity',value);
		    });
		    setScrollerProp($("#div_scrollButton"), true);
		    CheckIsEmptyCol(window.manifest.options.hiddenColumns);
	    }
	});
	formActivity();
    }, 1000);
    
    
    container_popups.on('change', '.positionTypes', function () {
	//setResetState(true);
	// Get the selected options of all positions
	var allSelected = $(".positionTypes").map(function () {
	    return $(this).val();
	}).get();
	
	// set all enabled
	$(".positionTypes option").removeAttr("disabled");
	//Disable selected options in other positions
	
	$(".positionTypes option:not(:selected)").each(function () {
	    var selectedText = $(this).val();
	    var selectedIndex = $.inArray(selectedText, allSelected);
	    if (selectedIndex >= 0 && selectedText.trim() != "") {
		$(this).attr('disabled', true);
	    }
	});
	onSortDrpSelectChange();
    });

    
});

function onSortDrpSelectChange() {
    var sortDrps = container_popups.find(".positionTypes");
    for (var i=1; i<sortDrps.length; i++) {
	var prevDrpVal = sortDrps.eq(i-1).val();
	var currDrpSlot = sortDrps.eq(i).parents(".sortable_slot");
	if (!prevDrpVal) {
	    currDrpSlot.css({"opacity":0.5});
	    currDrpSlot.find("select").attr("disabled", "disabled");
	    currDrpSlot.find("input").attr("disabled", "disabled");
	}
	else {
	    currDrpSlot.css({"opacity":1});
	    currDrpSlot.find("select").removeAttr("disabled");
	    currDrpSlot.find("input").removeAttr("disabled");
	}
    }
}

function formActivity(){
    formDataTable();
    setTimeout(function(){ alignPositions(); }, 50);
    setTimeout(function(){ alignPositions(); }, 100);
    setTimeout(function(){ alignPositions(); }, 200);
}

function showWarningPopup(warningMsg) {
    $(".warn_content").html(warningMsg);
    warningPopup.css({"visibility":"visible"});
    $("#generalBlocker").show();
}

function hideWarningPopup() {
    warningPopup.css({"visibility":"hidden"});
    $("#generalBlocker").hide();
}

function formBasicTemplate() {
    alignContan_Btn=$("<div class=\"alignContan_Btn\" /> ");
    container_layout = $("<div class=\"whole_tableLayout\"/>");
    container_options = $("<div id=\"sidebar_right\" />");
    container_popups = $("<div id=\"container_popup\" />");
    container_table = $("<div id=\"main_left\" />")
    activityHolder.append(container_layout);
    container_layout.append(container_options).append(container_table);
    activityHolder.append(container_popups);
    popupDraggable = new Drag(container_popups);
}

function formOptions() {
    
    if (container_options && manifest) {
	var options = manifest.options;
	if (options.isSortable) {
	    var btn_sort = $("<div id=\"btn_sort\" class=\"optionBtn\" />");
	    btn_sort.html(options.sortButtonText);
	    container_options.append(btn_sort);
	    btn_sort.bind(eventType, function(){ formPopupWindow("sort"); });
	    
	}
	
	if (options.isHideShowColumns) {
	    var btn_hideShow = $("<div id=\"btn_hideShow\" style=\"width:175px;\" class=\"optionBtn\" />");
	    btn_hideShow.html(options.hideShowColumnsButtonText);
	    container_options.append(btn_hideShow);
	    btn_hideShow.bind(eventType, function(){ formPopupWindow("hideShow"); });
	}
	
	if (options.isShowStatisticalValues) {
	    var btn_showStatistics = $("<div id=\"btn_showStatistics\" class=\"optionBtn\" />");
	    btn_showStatistics.html(options.ShowStatisticalValuesButtonText);
	    container_options.append(btn_showStatistics);
	    btn_showStatistics.bind(eventType, function(){ formPopupWindow("showStatistics"); });
	}
    }
}

function formDataTable(){
    var maxColumns = 8;
    var maxRows = 500;
    if (container_table && manifest) {
	container_table.children().remove();
	var fontSize = getFontSize();
	var dataTable = $("<table id=\"dataTable\" style=\"font-size:"+fontSize+"px;\" />");
	var headerTable = $("<table id=\"headerTable\" style=\"font-size:"+fontSize+"px;\" />");
	var columns = manifest.columns;
	var hiddenCols = manifest.options.hiddenColumns ? manifest.options.hiddenColumns : [];
	sortAllColumns();
	for (var r=-1; r<Math.min(manifest.data.length, maxRows); r++) {
	    var dataRow = $("<tr />");
	    if (r == -1) {
		for (var c=-1; c<Math.min(manifest.columns.length, maxColumns); c++) {
		    if (hiddenCols.indexOf(c) < 0 || c == -1) {
			var header = "";
			if (c == -1) header = "";
			else header = manifest.columns[c].name;
			dataRow.append("<th class=\"col_"+c+"\">"+header+"</th>");
		    }
		}
		var headerClone = dataRow.clone();
		headerTable.append(headerClone);
	    }
	    else {
		var rowData = manifest.data[r];
		for (var c=-1; c<Math.min(manifest.columns.length, maxColumns); c++) {
		    var cellData = c < rowData.length ? rowData[c] : "";
		    if (c == -1) dataRow.append("<td>"+(r+1)+"</td>");
		    else if (hiddenCols.indexOf(c) < 0) dataRow.append("<td>"+rowData[c]+"</td>");
		}
	    }
	    dataTable.append(dataRow);
	}
	var div_tableHeader = $("<div id=\"div_tableHeader\" />");
	var div_tableBody = $("<div id=\"div_tableBody\" class=\"scrollArea\" />");
	var div_scrollBar = $("<div id=\"div_scrollBar\" />");
	var div_scrollButton = $("<div id=\"div_scrollButton\" onclick=\"setResetState(true);\" />");
	div_tableHeader.append(headerTable);
	div_tableBody.append(dataTable);
	var tableBodyHolder = $("<div id=\"tableBodyHolder\" />");
	div_scrollBar.append(div_scrollButton);
	tableBodyHolder.append(div_tableBody).append(div_scrollBar);
	container_table.append(div_tableHeader).append(tableBodyHolder);
	scrollbarDraggable = new Drag($("#div_scrollButton"), {axis:"y"});
	scrollbarDraggable.OnDrag = function(){ setScrollerProp($("#div_scrollButton"), true); };
	$("#div_scrollButton").css({"top":scrollPos+"px"});
	setScrollerProp($("#div_scrollButton"), true);
	alignPositions();
    }
}

function sortAllColumns() {
    var data = JSON.parse(manifest_copy_str).data;
    if (manifest)
	if (data && manifest.options.sortBy)
	    for (var s=manifest.options.sortBy.length-1; s>=0; s--)
		sortSingleColumn(data, manifest.options.sortBy[s]);
    manifest.data = data;
}

function sortSingleColumn(data, sortInfo){
    for(var _i=0; _i<data.length; _i++){
	for(var _j=0; _j<data.length; _j++){
	    if (_i != _j) {
		var val_1 = data[_i][sortInfo.column];
		var val_2 = data[_j][sortInfo.column];
		var row_2_temp = data[_j];
		var result = val_1 < val_2;
		if ((sortInfo.sort.toLowerCase()[0] == "a" && result) || (sortInfo.sort.toLowerCase()[0] == "d" && !result)){
		    data[_j] = data[_i];
		    data[_i] = row_2_temp;
		}
	    }
	}
    }
}

function alignPositions() {
    var fontSizeMultiplier=1;
    var isLoopThrough = false;
    do{
	var borderWidth = 1;
	var addWidth = 0;
	var activityHolderHeight = Math.floor($(".contentFeedback").position().top-$("#activityHolder").position().top)-5;
	
	activityHolder.css({"height":activityHolderHeight+"px"});
	
	var fontSize = getFontSize(fontSizeMultiplier);
	
	var div_tableHeader = $("#div_tableHeader");
	var div_tableBody = $("#div_tableBody");
	var table_header = div_tableHeader.children("table");
	var table_body = div_tableBody.children("table");
	var divScrollBar = $("#div_scrollBar");
	
	table_header.css({"font-size":fontSize+"px"});
	table_body.css({"font-size":fontSize+"px"});
	
	var td_actual_tableHeader = div_tableBody.find("tr").eq(0);
	var h_left = div_tableBody.position().left+borderWidth;
	div_tableHeader.css({"left":h_left+"px"});
	var headerTDs = table_header.find("th");
	var bodyTDs = table_body.find("tr").eq(0).find("th");
	for(var h=0; h<headerTDs.length; h++)
	    headerTDs.eq(h).css({"width":(bodyTDs.eq(h).outerWidth()+addWidth)+"px"});
	var b_top_1 = table_body.find("tr").eq(0).height()+2;
	var b_top = div_tableHeader.height();
	var b_height = activityHolderHeight-b_top;
	
	divScrollBar.css({"height":(b_height-45)+"px"});
	div_tableBody.css({"margin-top":b_top+"px", "height":(b_height-45)+"px"});
	table_body.css({"margin-top":-b_top_1+"px"});
	table_header.css({"width":table_body.outerWidth()+"px"});
	
	fontSizeMultiplier -= 0.1;
	isLoopThrough = Math.abs(divScrollBar.position().top - b_top) > 5 && fontSizeMultiplier > 0;
    }
    while (isLoopThrough);
    setScrollerProp($("#div_scrollButton"), false);
}

function setScrollerProp(scrollButton,readScroller) {
    var aniTime = 100;
    var scrollbar = scrollButton.parent();
    var referenceArea = scrollbar.parent().find(".scrollArea");
    var innerArea = referenceArea.children().eq(0);
    if (innerArea.height() > referenceArea.outerHeight(true)) {
	var hiddenArea = innerArea.height() - referenceArea.outerHeight(true);
	var scrollBtnMaxTop = scrollbar.height()-scrollButton.outerHeight(true);
	var posPerc = (parseFloat(scrollButton.css("top").replace("px","")))/scrollBtnMaxTop;
	ScrollPercent=hiddenArea*posPerc;
	referenceArea.scrollTop(ScrollPercent);
	ScrollPercent=referenceArea.scrollTop();
	if (!readScroller) {
	    scrollButton.clearQueue();
	    var visiblePerc = referenceArea.outerHeight(true)/innerArea.height();
	    var proposedHeight = visiblePerc*scrollbar.height();
	    scrollButton.css({"height":proposedHeight+"px"});
	    proposedHeight = scrollButton.height();
	    var topAdj = (scrollButton.position().top+proposedHeight) - scrollbar.height();
	    if (topAdj > 0) {
		var newTop = scrollButton.position().top-topAdj;
		scrollButton.animate({"top":newTop+"px"}, aniTime);
	    }
	    var posPerc = referenceArea.scrollTop()/hiddenArea;
	    scrollButton.css({"top":(scrollBtnMaxTop*posPerc)+"px"});
	    if (visiblePerc >= 1) scrollbar.animate({"opacity":0},aniTime);
	    else scrollbar.animate({"opacity":1},aniTime);
	}
	scrollPos = scrollButton.position().top;
    }
    else scrollbar.animate({"opacity":0},aniTime);
}

function getFontSize(fontSizeMultiplier){
    if (!fontSizeMultiplier) fontSizeMultiplier = 1;
    return Math.floor(Math.max(container_layout.width()/70, 2)*fontSizeMultiplier);
}

function formPopupWindow(popupFor) {
    setResetState(true)
    if (container_popups && manifest) {
	var alreadyOpened = container_popups.children().eq(0);
	if (alreadyOpened) {
	    if(alreadyOpened.length)
		alreadyOpened = alreadyOpened.attr("id").toString().replace("popup_", "");
	    else alreadyOpened = "";
	}
	if (alreadyOpened) return;
	closePopup();
	var popupContent = $("<div id=\"popup_"+popupFor+"\" class=\"popup\" />");
	var innerContent;
	container_popups.css({"visibility":"visible"});
	var selectionInDrp = [];
	if(popupFor == "sort"){
	    innerContent = $("<div />");
	    innerContent.append("<div class=\"div_sort_sortBy\">Sort by</div>");
	    
	    var sortableCount = 3;
	    for (var i=0; i<sortableCount; i++ ) {
		var div_sortableSlot = $("<div id=\"sortable_slot_"+i+"\" class=\"sortable_slot\" />");
		if (i > 0) div_sortableSlot.append("<div class=\"div_sort_thenBy\">...then by</div>");
		var selectionBoxId = "sort_select_"+i;
		var selectionBox = "<select class=\"positionTypes\" id=\""+selectionBoxId+"\">";
		for(var c=0; c<manifest.columns.length; c++){
		    var optionContent = "";
		    if (c >= 0) optionContent = manifest.columns[c].name.replace(/\<br.*\>/g," ");
		    selectionBox += "<option value=\""+c+"\">"+optionContent+"</option>";
		}
		selectionBox += "</select>";
		div_sortableSlot.append(selectionBox);
		
		div_sortableSlot.append(createRadioButtonGroup([{name:"Ascending", value:"asc", checked:true}, {name:"Descending", value:"desc", checked:false}], "sort_col_"+i, 2));
		
		var sortBy = manifest.options.sortBy;
		if (i < sortBy.length) {
		    var sortOption = sortBy[i];
		    selectionInDrp.push({name:"#"+selectionBoxId, index:sortOption.column});
		    div_sortableSlot.find("#"+selectionBoxId+" option[value='"+sortOption.column+"']").attr("selected", "selected");
		    if (!sortOption.sort || sortOption.sort[0] != "a") {
			div_sortableSlot.find(".table_radioButtonGroup").eq(i).find("input[value='desc']").attr("checked", "checked");
		    }
		}
		else selectionInDrp.push({name:"#"+selectionBoxId, index:-1});
		innerContent.append(div_sortableSlot);
	    }
	    innerContent.append("<div class=\"div_sortBtnsHolder\"><div id=\"div_Reset_sortBy\" class=\"btn_type1\">Reset</div><div id=\"btn_doSort\" class=\"btn_type1\">Sort</div></div>");
	}
	else if(popupFor == "hideShow"){
	    innerContent = $("<div />");
	    var hiddenCols = manifest.options.hiddenColumns;
	    for(var c=0; c<manifest.columns.length; c++){
		var columnName = manifest.columns[c].name.replace(/\<br.*\>/g," ");
		var optionAsStr = "<div class=\"div_hideShowEntries\">";
		optionAsStr += "<div class=\"div_hideShowEntry\">"+columnName+"</div><div value=\""+c+"\" class=\"btn_showHideEye "+(hiddenCols.indexOf(c) >= 0 ? "hiddenEye" : "")+"\" />";
		optionAsStr += "</div>";
		innerContent.append(optionAsStr);
	    }
	    innerContent.append("<div id=\"btn_doShowHide\" class=\"btn_type1\">Hide/Show</div>");
	}
	else if(popupFor == "showStatistics"){
	    innerContent = $("<div />");
	    innerContent.append("<div class=\"div_datasetSelectionHead\">Dataset selected:</div>");
	    var optionAsStr = "<select class=\"div_datasetType\">";
	    for(var c=0; c<manifest.columns.length; c++){
		var columnName = "";
		if (c >= 0) {
		    columnName = manifest.columns[c].name.replace(/\<br.*\>/g," ");
		    if(manifest.columns[c].type == "quantity")
			optionAsStr += "<option value=\""+c+"\">"+columnName+"</option>";
		}
		else optionAsStr += "<option value=\""+c+"\">"+columnName+"</option>";
	    }
	    optionAsStr += "</select>";
	    innerContent.append(optionAsStr);
	    innerContent.append(createRadioButtonGroup([{name:"Complete dataset", value:"comp", checked:true}, {name:"Partial Dataset:", value:"part", checked:false}], "statistics_dataset", 1));
	    innerContent.append("<div class=\"div_partialDataRange\">from row <input type=\"text\" id=\"dataset_from\" class=\"datasetTxtBox\" /> through row <input type=\"text\" id=\"dataset_to\" class=\"datasetTxtBox\" /></div>");
	    innerContent.append("<div id=\"btn_doGetValues\" class=\"btn_type1\">Get Values</div>");
	    
	    var statisticsDisplay = "<table>";
	    for (var s=0; s<statisticsToDisplay.length; s++) {
		var text = statisticsToDisplay[s];
		var name = text.replace(/\s/g, "").toLowerCase();
		statisticsDisplay += "<tr>";
		statisticsDisplay += "<td id=\"name_"+name+"\">"+text+"</td>";
		statisticsDisplay += "<td>:</td>";
		statisticsDisplay += "<td id=\"result_"+name+"\"></td>";
		statisticsDisplay += "</tr>";
	    }
	    statisticsDisplay += "</table>";
	    innerContent.append(statisticsDisplay);
	}
	
	popupContent.append(innerContent);
	container_popups.append(popupContent);
	container_popups.append($("<div class=\"btn_popupClose\" />"));
	
	for (var _s=0; _s<selectionInDrp.length; _s++) {
	    var selection = selectionInDrp[_s];
	    $(selection.name)[0].selectedIndex = selection.index;
	}
	
	//$('.positionTypes,.div_datasetType').val("");
	$("#div_Reset_sortBy").bind(eventType, function(){ resetSorBox(); });
	$("#btn_doSort").bind(eventType, function(){ doSort(); });
	$(".btn_showHideEye").bind(eventType, function(){ toggleShowHide($(this)); });
	$("#btn_doShowHide").bind(eventType, function(){ doShowHide(); });
	$("#btn_doGetValues").bind(eventType, function(){ doGetValues(); });
	$(".btn_popupClose").bind(eventType, function(){ closePopup(); });
	$('.td_radioButton,.td_radioLabel').bind(eventType,function(){EmptyTxtFeild();});
	var popupPos = container_popups.position();
	var left = popupPos.left;
	var top = popupPos.top;
	if (!isFirstPopupOpen) {
	    if(container_options.children().length >= 2){
		var lastButton = container_options.children().eq(container_options.children().length-2);
		top = container_table.offset().top;
	    }
	    left = container_table.width();
	    container_popups.css({"left":left+"px", "top":top+"px"});
	    isFirstPopupOpen = true;
	}
	popupDraggable.setPosition({x:left, y:top}, container_popups);
	$(".optionBtn").addClass("inactive");
	$("#btn_"+popupFor).addClass("active").removeClass("inactive");
	simpleKeypad.loadControls();
	onSortDrpSelectChange();
    }
}

function createRadioButtonGroup(nameValues,groupName, trEvery) {
    var html = "<table class=\"table_radioButtonGroup\"><tr>";
    for(var i=0; i<nameValues.length; i++){
	var value = nameValues[i];
	if (i%trEvery == 0 && i != 0)
	    html += "</tr><tr>";
	var radName = "chk_"+groupName+"_"+i;
	html += "<td class=\"td_radioButton\"><input type=\"radio\" id=\""+radName+"\" name=\""+groupName+"\" value=\""+value.value+"\" "+(value.checked ? "checked=\"checked\"" : "")+"></td>";
	html += "<td class=\"td_radioLabel\"><label for=\""+radName+"\" >"+value.name+"</label></td>";
    }
    html += "</tr>";
    return html;
}

function doSort() {
    scrollPos = 0;
    var sortBy = [];
    var firstSortBy = parseInt($("#sort_select_0 option:selected").attr("value"));
    var firstSortOrder = $("#chk_sort_col_0_0").prop("checked") ? "asc" : "desc";
    var secondSortBy = parseInt($("#sort_select_1 option:selected").attr("value"));
    var secondSortOrder = $("#chk_sort_col_1_0").prop("checked") ? "asc" : "desc";
    var thirdSortBy = parseInt($("#sort_select_2 option:selected").attr("value"));
    var thirdSortOrder = $("#chk_sort_col_2_0").prop("checked") ? "asc" : "desc";
    
    var isRepeated = false;
    if (firstSortBy >= 0) {
	if (firstSortBy == secondSortBy) isRepeated = true;
	else if (firstSortBy == thirdSortBy) isRepeated = true;
    }
    if (secondSortBy >= 0) {
	if (secondSortBy == firstSortBy) isRepeated = true;
	else if (secondSortBy == thirdSortBy) isRepeated = true;
    }
    if (thirdSortBy >= 0) {
	if (thirdSortBy == firstSortBy) isRepeated = true;
	else if (thirdSortBy == secondSortBy) isRepeated = true;
    }
    if (isRepeated){
	showWarningPopup("The sorting column can only be selected once.");
	return;
    }
    if (firstSortBy >= 0) sortBy.push({column: firstSortBy, sort:firstSortOrder});
    if (secondSortBy >= 0) sortBy.push({column: secondSortBy, sort:secondSortOrder});
    if (thirdSortBy >= 0) sortBy.push({column: thirdSortBy, sort:thirdSortOrder});
    manifest.options.sortBy = sortBy;
    formDataTable();
    setResetState(true);
}
function resetSorBox() {
    $('#sort_select_0').val('');
    $('#chk_sort_col_0_0').prop('checked',true);
    $('#chk_sort_col_0_1').prop('checked',false);
    $('#sort_select_1').val('');
    $('#chk_sort_col_1_0').prop('checked',true);
    $('#chk_sort_col_1_1').prop('checked',false);
    $('#sort_select_2').val('');
    $('#chk_sort_col_2_0').prop('checked',true);
    $('#chk_sort_col_2_1').prop('checked',false);
    $('.positionTypes').find('option').attr('disabled', false);
    onSortDrpSelectChange();
}
function doShowHide(){
    scrollPos = 0;
    var showHideButtons = $(".btn_showHideEye");
    var hiddenCols = [];
    for(var c=0; c<showHideButtons.length; c++)
	if(showHideButtons.eq(c).hasClass("hiddenEye"))
	    hiddenCols.push(c);
    manifest.options.hiddenColumns = hiddenCols;
    formDataTable();
    setResetState(true);
    CheckIsEmptyCol(hiddenCols);
}
function CheckIsEmptyCol(col) {
    var cols=col.length;
    
    if(manifest.columns.length==cols) {
	
	$('#headerTable tr>th:first-child').css({'border-right':'0px solid #9ECE61'});
    $('#div_tableHeader').css({'border-right':'0px solid #000'});
    }
    else {
	$('#headerTable tr>th:first-child').css({'border-right':'2px solid #032D19'});
    $('#div_tableHeader').css({'border-right':'1px solid #000'});
    }
}
function toggleShowHide(showHideIcon) {
    var hiddenClass = "hiddenEye";
    if (!showHideIcon.hasClass(hiddenClass)) showHideIcon.addClass(hiddenClass);
    else showHideIcon.removeClass(hiddenClass)
}

function medianOf(sortedValues) {
    var medianValue="";
    if (sortedValues.length) {
	var midIndex = Math.floor(sortedValues.length/2);
	if (sortedValues.length % 2 == 0)
	    medianValue = (sortedValues[midIndex-1]+sortedValues[midIndex])/2;
	else medianValue = sortedValues[midIndex];
    }
    return medianValue;
}

function doGetValues(){
    try {
    var maxDecimalPlaces = 3;
    var selectedDataset = parseInt($(".div_datasetType option:selected").attr("value"));
    var dataRange = [];
    var isLimitExeeding = false;
    if($("#chk_statistics_dataset_0").prop("checked"))
	dataRange = [1, manifest.data.length];
    else {
	var fromVal = $("#dataset_from")[0].value.trim();
	var toVal = $("#dataset_to")[0].value.trim();
	if (fromVal && toVal) {
	    fromVal = parseInt(fromVal);
	    toVal = parseInt(toVal);
	    if (fromVal > manifest.data.length || toVal > manifest.data.length || fromVal <= 0 || toVal <= 0) isLimitExeeding = true;
	    else dataRange = [Math.min(fromVal, toVal),Math.max(fromVal, toVal)];
	}
    }
    
    if (selectedDataset >= 0 && dataRange.length == 2) {
	var sumValue = 0, meanValue = 0, medianValue=0, dataCount = 0, values = [], sortedValues=[], differences=[], differencesSquare=[], diffSum=0, diffSqSum=0, variance = 0;
	var Q1 = "", Q3 = "", IQR="";
	for (var d=Math.min(dataRange[0], dataRange[1]); d<=Math.max(dataRange[0], dataRange[1]); d++) {
	    var dataValue = manifest.data[d-1][selectedDataset];
	    if ((typeof dataValue)== "string")
		dataValue = parseFloat(dataValue);
	    if (!isNaN(dataValue)) {
		sumValue += dataValue;
		values.push(dataValue);
		dataCount++;
	    }
	}
	meanValue = sumValue / dataCount;
	sortedValues = JSON.parse(JSON.stringify(values));
	for(var _i=0; _i<sortedValues.length; _i++){
	    for(var _j=0; _j<sortedValues.length; _j++){
		if (_i != _j) {
		    var val_1 = sortedValues[_i];
		    var val_2 = sortedValues[_j];
		    var val_2_temp = sortedValues[_j];
		    var result = val_1 < val_2;
		    if (result){
			sortedValues[_j] = sortedValues[_i];
			sortedValues[_i] = val_2_temp;
		    }
		}
	    }
	}
	
	for (var i=0; i<values.length; i++) {
	    var value=Math.abs(values[i]-meanValue);
	    diffSum += value;
	    differences.push(value);
	    var sqVal = Math.pow(value, 2);
	    diffSqSum += sqVal;
	    differencesSquare.push(sqVal);
	}
	variance = diffSqSum/values.length;
	
	medianValue = medianOf(sortedValues);
	
	var Q1Values = [], Q3Values = [];
	
	if (sortedValues.length >= 2){
	    for (var i=0; i<sortedValues.length; i++) {
		var value = sortedValues[i];
		if (value < medianValue)
		    Q1Values.push(value);
		else if (value > medianValue)
		    Q3Values.push(value);
	    }
	    
	    Q1 = medianOf(Q1Values);
	    Q3 = medianOf(Q3Values);
	    IQR = Q3-Q1;
	}
	
	for (var s=0; s<statisticsToDisplay.length; s++) {
	    var text = statisticsToDisplay[s];
	    var funcName = text.replace(/\s/g, "").toLowerCase();
	    var resultDisplayId = "result_"+funcName;
	    var labelDisplayId = "name_"+funcName;
	    var resultValue = "";
	    
	    if (funcName == "sum") resultValue = sumValue;
	    else if (funcName == "mean") resultValue = meanValue;
	    else if (funcName == "median") resultValue =medianValue;
	    else if (funcName == "mode") {
		var mapedModes = [];
		var modeMap = [];
		for(var i=0; i<values.length; i++){
		    var value = values[i], count=0;
		    if (mapedModes.indexOf(value) < 0)
			for(var j=0; j<values.length; j++) {
			    if (i != j) 
				if (value == values[j])
				    count++;
			}
		    
		    if (count >= 1)
			modeMap.push({mode:value, count:count});
		    mapedModes.push(value);
		}
		var modeValues = [];
		var maxCount = 0;
		for (var m=0; m<modeMap.length; m++) {
		    if(modeMap[m].count > maxCount){
			maxCount = modeMap[m].count;
			modeValues = [modeMap[m].mode];
		    }
		    else if (modeMap[m].count == maxCount) modeValues.push(modeMap[m].mode);
		}
		for (var i=0; i<modeValues.length; i++)
		    resultValue += (i!=0 ? ", " : "") + modeValues[i];
	    }
	    else if (funcName == "range") {
		if (sortedValues.length > 1)
		    resultValue = sortedValues[sortedValues.length-1]-sortedValues[0];
	    }
	    else if (funcName == "standarddeviation") resultValue = Math.sqrt(variance);
	    else if (funcName == "meanabsolutedeviation") resultValue = diffSum/values.length;
	    else if (funcName == "interquatilerange") resultValue = IQR;
	    else if (funcName == "firstquartileq1") resultValue = Q1;
	    else if (funcName == "thirdquartileq3") resultValue = Q3;
	    
	    var resultValueAsStr = resultValue.toString();
	    var decimalPos = resultValueAsStr.length - resultValueAsStr.indexOf(".")-1;
	    if (decimalPos < resultValueAsStr.length) {
		if (decimalPos > maxDecimalPlaces) resultValueAsStr = resultValue.toFixed(maxDecimalPlaces);
	    }
	    
	    if (funcName == "mode") {
		if (modeValues.length > 1)
		    $("#"+labelDisplayId).html("Modes");
		else $("#"+labelDisplayId).html("Mode");
	    }
	    
	    $("#"+resultDisplayId).html(resultValueAsStr);
	}
	var popup_pos = container_popups.position();
	var newPopup_pos = {x:popup_pos.left, y:popup_pos.top};
	popupDraggable.setPosition(newPopup_pos, container_popups);
	setResetState(true);
    }
    else if(selectedDataset < 0) showWarningPopup("Please select atleast one dataset.");
    else if (isLimitExeeding) showWarningPopup("Range of partial dataset you provided is incorrect. Please check.");
    else showWarningPopup("Please select the rows of the partial dataset.");
    } catch(e) {
    }
}

function closePopup(){
    container_popups.children().remove();
    container_popups.css({"visibility":"hidden"});
    $(".optionBtn").removeClass("inactive");
    $(".optionBtn.active").removeClass("active");
    popupDraggable.isTouchDown = false;
}
function EmptyTxtFeild() {
    $('.datasetTxtBox').val("");
}

function setResetState(state){
    
    if (state) $(".icoReset").css({"opacity":"1"});
    else $(".icoReset").css({"opacity":"0.5"});
}

function resetClick() {
    var resetOpacity = $(".icoReset").css("opacity");
    resetOpacity = resetOpacity ? resetOpacity : 0;
    resetOpacity = parseFloat(resetOpacity);
    if (resetOpacity > 0.9) {
	$(".icoReset").removeClass("icoResetMove");
	var to=setTimeout(function(){
	    clearInterval(to);
	    $(".icoReset").addClass("icoResetMove");
	    setResetState(false);
	},200);
	/*Rese here*/
	window.manifest = JSON.parse(manifest_copy_str);
	scrollPos = 0;
	formActivity();
	closePopup();
    }
}

function closeInstruction(isNoInst){
    if (instructionDivs) {
	instructionDivs.hide();
	if (isNoInst) $(".icoInfo").css({"opacity":0.5});
	else $(".icoInfo").css({"opacity":1});
    }
}

function showInstruction(){
    if ($("#IOContent").text().trim() == ""){
	closeInstruction(true);
	return;
    }
    if (instructionDivs) {
	instructionDivs.show();
	$(".icoInfo").css({"opacity":0.5});
    }
}

eventBroker = _({}).extend(require("chaplin/lib/event_broker"));
eventBroker.subscribeEvent("#doSave", function(state) {
    var states = {};
    states = {"activityName":activityName, "activityVersion":activityVersion, "manifest":manifest,"ScrollPercent":ScrollPercent,"div_scrollButtonPos":$("#div_scrollButton").position().top,"icoReset":$('.icoReset').css('opacity')};
    var message = {
	type : "state",
	data : JSON.stringify(states)
    };
    eventBroker.publishEvent("#save", message);
});
function saveFunction(){
    eventBroker.publishEvent("#doSave");
}










/////////////////////////////////////


manifest.js
==========

/*
    Usage:
        Maximum of 8 columns
        Type of columns - [category] and [quantity]
        sortBy - Template
            [
                {column:0, sort:"asc"},
                {column:2, sort:"desc"},
            ]
        hiddenColumns - Leave "Empty" to show all columns
*/

var manifest={
    activityName:"StatisticsSortingTable",
    instruction:"Instructions Placeholder",
    columns:[
        {name:"Year",type:"quantity"},
        {name:"State",type:"quantity"},
        {name:"Number of<br />People",type:"quantity"},
    ],
    data:[
            [96.3,1,70],
            [96.7,1,71],
            [96.9,1,74],
            [97.1,1,75],
            [97.2,1,64],
            [97.3,1,69],
            [97.4,1,70],
            [97.4,1,72],
            [97.5,1,70],
            [97.5,1,75],
            [97.6,1,74],
            [97.6,1,69],
            [97.7,1,77],
            [97.8,1,58],
            [97.8,1,74],
            [97.9,1,76],
            [98,1,78],
            [98,1,73],
            [98,1,67],
            [98,1,66],
            [98,1,64],
            [98,1,71],
            [98.1,1,72],
            [98.1,1,86],
            [98.2,1,72],
            [98.2,1,68],
            [98.2,1,70],
            [98.2,1,82],
            [98.3,1,84],
            [98.3,1,68],
            [98.4,1,71],
            [98.4,1,77],
            [98.4,1,78],
            [98.4,1,83],
            [98.5,1,66],
            [98.6,1,82],
            [98.6,1,84],
            [98.6,1,71],
            [98.6,1,77],
            [98.6,1,78],
            [98.7,1,83],
            [98.7,1,66],
            [98.8,1,70],
            [98.8,1,82],
            [99,1,81],
            [99,1,78],
            [99.1,1,80],
            [99.2,1,75],
            [99.4,1,81],
            [99.5,1,71],
            [96.4,2,83],
            [96.7,2,63],
            [97.2,2,75],
            [97.2,2,69],
            [97.6,2,75],
            [97.7,2,66],
            [97.7,2,68],
            [97.8,2,57],
            [97.8,2,61],
            [97.8,2,84],
            [97.9,2,61],
            [97.9,2,77],
            [97.9,2,62],
            [98,2,71],
            [98,2,69],
            [98,2,79],
            [98,2,76],
            [98.1,2,87],
            [98.2,2,78],
            [98.2,2,73],
            [98.2,2,64],
            [98.3,2,65],
            [98.3,2,73],
            [98.3,2,69],
            [98.4,2,57],
            [98.4,2,79],
            [98.4,2,81],
            [98.4,2,74],
            [98.6,2,82],
            [98.6,2,85],
            [98.6,2,86],
            [98.7,2,72],
            [98.7,2,79],
            [98.7,2,65],
            [98.7,2,82],
            [98.8,2,64],
            [98.8,2,70],
            [98.8,2,83],
            [98.8,2,89],
            [98.8,2,69],
            [98.8,2,73],
            [98.8,2,84],
            [98.9,2,76],
            [99,2,79],
            [99,2,81],
            [99.2,2,77],
            [99.2,2,66],
            [99.3,2,68],
            [99.4,2,77],
            [100,2,78],
            [96.3,1,70],
            [96.7,1,71],
            [96.9,1,74],
            [97.1,1,75],
            [97.2,1,64],
            [97.3,1,69],
            [97.4,1,70],
            [97.4,1,72],
            [97.5,1,70],
            [97.5,1,75],
            [97.6,1,74],
            [97.6,1,69],
            [97.7,1,77],
            [97.8,1,58],
            [97.8,1,74],
            [97.9,1,76],
            [98,1,78],
            [98,1,73],
            [98,1,67],
            [98,1,66],
            [98,1,64],
            [98,1,71],
            [98.1,1,72],
            [98.1,1,86],
            [98.2,1,72],
            [98.2,1,68],
            [98.2,1,70],
            [98.2,1,82],
            [98.3,1,84],
            [98.3,1,68],
            [98.4,1,71],
            [98.4,1,77],
            [98.4,1,78],
            [98.4,1,83],
            [98.5,1,66],
            [98.6,1,82],
            [98.6,1,84],
            [98.6,1,71],
            [98.6,1,77],
            [98.6,1,78],
            [98.7,1,83],
            [98.7,1,66],
            [98.8,1,70],
            [98.8,1,82],
            [99,1,81],
            [99,1,78],
            [99.1,1,80],
            [99.2,1,75],
            [99.4,1,81],
            [99.5,1,71],
            [96.4,2,83],
            [96.7,2,63],
            [97.2,2,75],
            [97.2,2,69],
            [97.6,2,75],
            [97.7,2,66],
            [97.7,2,68],
            [97.8,2,57],
            [97.8,2,61],
            [97.8,2,84],
            [97.9,2,61],
            [97.9,2,77],
            [97.9,2,62],
            [98,2,71],
            [98,2,69],
            [98,2,79],
            [98,2,76],
            [98.1,2,87],
            [98.2,2,78],
            [98.2,2,73],
            [98.2,2,64],
            [98.3,2,65],
            [98.3,2,73],
            [98.3,2,69],
            [98.4,2,57],
            [98.4,2,79],
            [98.4,2,81],
            [98.4,2,74],
            [98.6,2,82],
            [98.6,2,85],
            [98.6,2,86],
            [98.7,2,72],
            [98.7,2,79],
            [98.7,2,65],
            [98.7,2,82],
            [98.8,2,64],
            [98.8,2,70],
            [98.8,2,83],
            [98.8,2,89],
            [98.8,2,69],
            [98.8,2,73],
            [98.8,2,84],
            [98.9,2,76],
            [99,2,79],
            [99,2,81],
            [99.2,2,77],
            [99.2,2,66],
            [99.3,2,68],
            [99.4,2,77],
            [100,2,78],
            [96.3,1,70],
            [96.7,1,71],
            [96.9,1,74],
            [97.1,1,75],
            [97.2,1,64],
            [97.3,1,69],
            [97.4,1,70],
            [97.4,1,72],
            [97.5,1,70],
            [97.5,1,75],
            [97.6,1,74],
            [97.6,1,69],
            [97.7,1,77],
            [97.8,1,58],
            [97.8,1,74],
            [97.9,1,76],
            [98,1,78],
            [98,1,73],
            [98,1,67],
            [98,1,66],
            [98,1,64],
            [98,1,71],
            [98.1,1,72],
            [98.1,1,86],
            [98.2,1,72],
            [98.2,1,68],
            [98.2,1,70],
            [98.2,1,82],
            [98.3,1,84],
            [98.3,1,68],
            [98.4,1,71],
            [98.4,1,77],
            [98.4,1,78],
            [98.4,1,83],
            [98.5,1,66],
            [98.6,1,82],
            [98.6,1,84],
            [98.6,1,71],
            [98.6,1,77],
            [98.6,1,78],
            [98.7,1,83],
            [98.7,1,66],
            [98.8,1,70],
            [98.8,1,82],
            [99,1,81],
            [99,1,78],
            [99.1,1,80],
            [99.2,1,75],
            [99.4,1,81],
            [99.5,1,71],
            [96.4,2,83],
            [96.7,2,63],
            [97.2,2,75],
            [97.2,2,69],
            [97.6,2,75],
            [97.7,2,66],
            [97.7,2,68],
            [97.8,2,57],
            [97.8,2,61],
            [97.8,2,84],
            [97.9,2,61],
            [97.9,2,77],
            [97.9,2,62],
            [98,2,71],
            [98,2,69],
            [98,2,79],
            [98,2,76],
            [98.1,2,87],
            [98.2,2,78],
            [98.2,2,73],
            [98.2,2,64],
            [98.3,2,65],
            [98.3,2,73],
            [98.3,2,69],
            [98.4,2,57],
            [98.4,2,79],
            [98.4,2,81],
            [98.4,2,74],
            [98.6,2,82],
            [98.6,2,85],
            [98.6,2,86],
            [98.7,2,72],
            [98.7,2,79],
            [98.7,2,65],
            [98.7,2,82],
            [98.8,2,64],
            [98.8,2,70],
            [98.8,2,83],
            [98.8,2,89],
            [98.8,2,69],
            [98.8,2,73],
            [98.8,2,84],
            [98.9,2,76],
            [99,2,79],
            [99,2,81],
            [99.2,2,77],
            [99.2,2,66],
            [99.3,2,68],
            [99.4,2,77],
            [100,2,78]
    ],
    options:{
        isSortable:true,
        sortButtonText:"Sort Table",
        sortBy:[],
        
        isHideShowColumns:true,
        hideShowColumnsButtonText:"Hide/Show columns",
        hiddenColumns:[],
        
        isShowStatisticalValues:true,
        ShowStatisticalValuesButtonText:"Show Statistical Values"
    }
};






