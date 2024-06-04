<?php
/*
Copyright 2006-2018 Felix Rudolphi and Lukas Goossen
open enventory is distributed under the terms of the GNU Affero General Public License, see COPYING for details. You can also find the license under http://www.gnu.org/licenses/agpl.txt

open enventory is a registered trademark of Felix Rudolphi and Lukas Goossen. Usage of the name "open enventory" or the logo requires prior written permission of the trademark holders.

This file is part of open enventory.

open enventory is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

open enventory is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with open enventory.  If not, see <http://www.gnu.org/licenses/>.
*/
// TCI

// Khoi: for troubleshooting running this file in PHP command line
// require_once(dirname(__FILE__, 2).'/lib_supplier_scraping.php');

$GLOBALS["code"]="TCI";
$code=$GLOBALS["code"];

$GLOBALS["suppliers"][$code]=array(
	"code" => $code,
	"name" => "TCI",
	"logo" => "tci.gif",
	"height" => 44,
	"vendor" => true,
	"hasPriceList" => 3,
	"alwaysProcDetail" => true,
	"searchTypeCode" => array("cas_nr" => 0, "emp_formula" => 4, "molecule_name" => 2),

	// "init" => create_function('', getFunctionHeader().'
	// 	$suppliers[$code]["urls"]["server"] = "https://www.tcichemicals.com"; // startPage  // new TCI website: from 2020-05-10
	// 	$suppliers[$code]["urls"]["search"] = $urls["server"]."/US/en/search/?text=";  // new TCI website: from 2020-05-10
	// 	$suppliers[$code]["urls"]["search2"] = "&searchWord=";
	// 	$suppliers[$code]["urls"]["search3"] = "&PRODUCT-MAIN-BTN.x=18&PRODUCT-MAIN-BTN.y=22&mode=0";
	// 	$suppliers[$code]["urls"]["detail"] = $urls["server"]."/US/en/p/";
	// 	$suppliers[$code]["urls"]["sds"] = $urls["server"]."US/en/documentSearch/productSDSSearchDoc";
	// 	$suppliers[$code]["urls"]["sds2"] = "&lang=";
	// 	$suppliers[$code]["urls"]["startPage"] = $urls["server"];
	// '),
	"init" => function () {
		eval(getFunctionHeader());
		$suppliers[$code]["urls"]["server"] =  $GLOBALS["suppliers"][$code]["regionPath"]("https://www.tcichemicals.com");
		$suppliers[$code]["urls"]["search"] = $urls["server"]."search/?text=";  // new TCI website: from 2020-05-10
		$suppliers[$code]["urls"]["detail"] = $urls["server"]."p/";
		$suppliers[$code]["urls"]["sds"] = $urls["server"]."documentSearch/productSDSSearchDoc";
		$suppliers[$code]["urls"]["startPage"] = $urls["server"];
	},

	"regionPath" => function($url) use ($noConnection, $default_http_options) {
		if (empty($url)) {
			return $noConnection;
		}
		$my_http_options = $default_http_options;
		$my_http_options["redirect"] = maxRedir;
		$response = oe_http_get($url,$my_http_options);
		if ($response===FALSE) {
			return $noConnection;
		}
		$body = $response->getBody();
		// Get the content of the head tag
		cutRange($body,"<head>","</head>");  // new TCI website: from 2020-05-10
		// print_r($body);

		// Look like this would provide a full url for the access region: '<link rel="canonical" href="https://www.tcichemicals.com/US/en/">'
		preg_match("/(?ims)<link[^>]*?rel=\"canonical\"[^>]*?href=\"(.*?)\"/",$body,$completeURL);  // new TCI website: from 2020-05-10
		// print_r($completeURL[1]);
		return $completeURL[1];
	},

	"requestResultList" => function ($query_obj) {
		eval(getFunctionHeader());
		$retval["method"] = "url";
		$search_type_code = $self["searchTypeCode"][$query_obj["crits"][0]];
		// $retval["action"] = $urls["search"].$search_type_code.$urls["search2"].urlencode($query_obj["vals"][0][0]).$urls["search3"];
		$retval["action"] = $urls["search"].urlencode($query_obj["vals"][0][0]);  // new TCI website: from 2020-05-10
		return $retval;
	},

	// Khoi: for creating link to Detail Page on Supplier page as well as for Create New data
	// Khoi: anonymous function does not work for some reason.
	"getDetailPageURL" => create_function('$catNo', getFunctionHeader().'
		// Khoi: new TCI website: from 2020-05-10, without referrer, the link redirects to homepage sometimes
		return $urls["detail"] . $catNo . "?referrer=tcichemicals.com";
		// return $urls["detail"] . $catNo;
	'),

	"getInfo" => create_function('$catNo', getFunctionHeader().'
		$url = $self["getDetailPageURL"]($catNo);
		if (empty($url)) {
			return $noConnection;
		}
		$my_http_options = $default_http_options;
		$my_http_options["redirect"] = maxRedir;
		$response = oe_http_get($url,$my_http_options);
		if ($response===FALSE) {
			return $noConnection;
		}
		return $self["procDetail"]($response,$catNo);
	'),

	"getHitlist" => function ($searchText, $filter, $mode="ct", $paramHash=array()) {
		eval(getFunctionHeader());
		$search_type_code=$self["searchTypeCode"][$filter];
		$url=$urls["search"].urlencode($searchText);   // new TCI website: from 2020-05-10
		$my_http_options=$default_http_options;
		$my_http_options["redirect"]=maxRedir;
		$response=oe_http_get($url,$my_http_options);
		if ($response===FALSE) {
			return $noConnection;
		}
		return $self["procHitlist"]($response, $searchText);   // new TCI website: from 2020-05-10
	},

	"procDetail" => create_function('&$response, $catNo=""', getFunctionHeader().'
		$body=@$response->getBody();
		$body=str_replace(array("&nbsp;","&middot;"),array(" ","*"),$body);
		// cutRange($body,"<h1 id=\"page-title\"","id=\"side\"");
		cutRange($body,"<div class=\"main__inner-wrapper\"","<footer>");  // new TCI website: from 2020-05-10
		// print_r($body);

		$result=array();
		$result["catNo"]=$catNo; // may be overwritten later
		$result["price"]=array();
		$result["molecule_property"]=array();

		// Khoi: grab the main name from the h1 tag of the detail page
		preg_match("/(?ims)<h1[^>]*name[^>]*>(.*?)<\/h1>/",$body,$name_data);  // new TCI website: from 2020-05-10
		// $result["molecule_names_array"]=array(fixTags($name_data[1]));
		// Khoi: grab all the synonyms as well:
		preg_match("/(?ims)<div class=\"code\">\s*?<ul>\s*?<li>(.*?)<\/li>\s*?<\/ul>/",$body,$synonyms);
		$result["molecule_names_array"] = array_merge(
			array(fixTags($name_data[1])),    // main name in the title
			preg_split("/<\/li>\s*?<li>/", $synonyms[1])    // list of synonyms in each "<li></li>" tag
		);

		preg_match_all("/(?ims)<img\s+src=\"\S*\/(\S+)\.png\"[^>]*?alt=\"Pictogram\"/",$body,$ghs_syms,PREG_PATTERN_ORDER);
		$tci_ghs_lookup = [
			"caution" => "7",     // GHS07: Harmful
			"person" => "8",     // GHS08: Health Hazard
			"fire1" => "2",     // GHS02: FLammable
			"fire2" => "3",     // GHS03: Oxidizing
			"danger" => "6",     // GHS06: Toxic
			"hands" => "5",     // GHS05: Corrosive
			"tree" => "9",     // GHS09: Environmental Hazard
		];
		if (count($ghs_syms) && count($ghs_syms[1])) {
			$ghs_list = array();
			foreach ($ghs_syms[1] as $word) {
				$ghs_list[] = $tci_ghs_lookup[$word];
			}
			$ghs_syms=join(",GHS0", $ghs_list);
			// $ghs_syms=join(",GHS0", $ghs_syms[1]);
			if (!isEmptyStr($ghs_syms)) {
				$result["safety_sym_ghs"]="GHS0".$ghs_syms; // add prefix while joining
			}
		}

		$safety=array();
		// preg_match_all("/(?ims)<[^>]+class\=\"[^\"]*code[^\"]*\"[^>]*>(.*?):/",$body,$phrases,PREG_PATTERN_ORDER);

		// Khoi: regex explain: https://regex101.com/r/cecocn/1/
		preg_match("/(?ims)<td>Signal Word.+?<\/table>/",$body,$phrases);
		$phrases=$phrases[0];
		// print_r($phrases);

		if (preg_match_all("/(?ims)(R|S|H|P)\s*(\d[\d\/\+\sHPRS]*)/",$phrases,$phrase_data, PREG_SET_ORDER)) foreach ($phrase_data as $phrase_datum) {
				$safety[ $phrase_datum[1] ][]=str_replace(array($phrase_datum[1]," "),"",$phrase_datum[2]);
		}

		$result["safety_r"]=@join("-",$safety["R"]);
		$result["safety_s"]=@join("-",$safety["S"]);
		$result["safety_h"]=@join("-",$safety["H"]);
		$result["safety_p"]=@join("-",$safety["P"]);


		preg_match_all("/(?ims)<table[^>]*class=\"productDetailTable\">(.*?)<\/table>/",$body,$data, PREG_PATTERN_ORDER);
		$data = join("\n", $data[1]);
		// print_r($data);

		// Khoi: does not work well with TCI new website (from 2020-05-10) because they missed some "</tr>" tag
		// preg_match_all("/(?ims)<tr[^>]*>(.*?)<\/tr>/",$data,$lines,PREG_PATTERN_ORDER);

		// Khoi: capture all two continuous "<td></td>"
		preg_match_all("/(?ims)(<td>.*?<\/td>)\s*?(<td>.*?<\/td>)/",$data,$lines,PREG_PATTERN_ORDER);

		$lines=$lines[0];
		// print_r($lines);

		if (is_array($lines)) foreach ($lines as $line) {
			// if (preg_match("/(?ims)<th[^>]*>(.*?)<\/th>\s*<td[^>]*>(.*?)<\/td>/",$line,$property)) {
			if (preg_match("/(?ims)<td[^>]*>(.*?)<\/td>\s*?<td[^>]*>(.*?)<\/td>/",$line,$property)) {
				$name=fixHtml($property[1]);
				$value=fixHtml($property[2]);
				// echo "NAME: $property[1]\n";
				// echo "VALUE: $property[2]\n\n";
				// echo "NAME: $name\n";
				// echo "VALUE: $value\n\n";

				switch ($name) {
				case "Product Number":
					$result["catNo"]=$value;
				break;
				// case "CAS Number":
				case "CAS RN":
					$result["cas_nr"]=$value;
				break;
				// case "UN Number":
				case "UN Number (DOT-AIR)":
					if (!isEmptyStr($value)) {
						$result["molecule_property"][]=array("class" => "UN_No", "source" => $code, "conditions" => $value);
					}
				break;
				case "EC Number":
					if (!isEmptyStr($value)) {
						$result["molecule_property"][]=array("class" => "EG_No", "source" => $code, "conditions" => $value);
					}
					break;
				case "Solubility in water":
					if (!isEmptyStr($value)) {
						$result["molecule_property"][]=array("class" => "Sol_water", "source" => $code,
															"value_high" => getNumber($value),
															"conditions" => ($result["molecule_property"]["value_high"] == "" ? $value : ""));
					}
					break;
				case "Specific rotation [a]20/D":
					if (!isEmptyStr($value)) {
						$result["molecule_property"][]=array("class" => "rotation_20", "source" => $code,
															// "value_high" => getNumber($value),
															"molecule_property_comment" => $value);
					}
					break;
				case "SG":
				case "Specific Gravity (20/20)":
					$result["density_20"]=getNumber($value);
				break;
				case "Refractive Index":
					$result["n_20"]=getNumber($value);
				break;
				case "Melting Point":
					list($result["mp_low"],$result["mp_high"])=getRange($value);
				break;
				case "Boiling Point":
					list($result["bp_low"],$result["bp_high"])=getRange($value);
				break;
				// case "flp":
				case "Flash point":
					$result["molecule_property"][]=array("class" => "FP", "source" => $code, "value_high" => getNumber($value), "unit" => "°C", );
				break;
				case "Packing Group (TCI-A)":
					$result["molecule_property"][]=array("class" => "packing_group", "source" => $code, "conditions" => $value, );
				break;
				case "Signal Word":
					$result["safety_text"]=$value;
				break;
				case "Molecular Formula / Molecular Weight":
					list($result["emp_formula"],$result["mw"])=explode("=", preg_replace("/[\s_]/", "", $value));
				break;
				}
			}
		}

		// read prices
		$result["price"]=$self["parsePriceList"]($body);

		$result["supplierCode"]=$code;
		return $result;
	'),

	"procHitlist" => function (&$response, $catNo) {
		eval(getFunctionHeader());
		$body=@$response->getBody();
		$body=str_replace(array("&nbsp;"),array(" "),$body);
		// cutRange($body,"id=\"main\"","id=\"tabPanel02\"");
		cutRange($body,"<div class=\"yCmsComponent search-list-page-right-result-list-component\"","<div id=\"pickupTitle\"");  // new TCI website: from 2020-05-10
		if (stripos($body,"The result is over")!==FALSE) { // too many results
			return $noConnection;
		}
		// new TCI website: from 2020-05-10, this is to find the Sort option, only showed if "Products" are found (not "documents", "content", etc.)
		if (stripos($body,"id=\"PLPSort\"") === FALSE) { // no results at all
			return $noResults;
		}

		$results=array();
		// if (stripos($body,"each result of")===FALSE) {
		// 	$results[0]=$self["procDetail"]($response);
		// 	extendMoleculeNames($results[0]);
		// 	$results[0]=array_merge($results[0],array("name" => $results[0]["molecule_name"], "supplierCode" => $code, ) );
		// }
		// else {

		// Match each result section, regex explation: https://regex101.com/r/BK6anU/2/
		// print_r($body);
		preg_match_all("/(?ims)<div[^>]*class=\"[^>]*product-description-wrap.+?<\/form>/",$body,$manyLines,PREG_SET_ORDER);    // new TCI website: from 2020-05-10
		// print_r(count($manyLines));
		for ($b=0;$b<count($manyLines);$b++) {
			// Search for "catNo" and "Name", regex explanation: https://regex101.com/r/572X9O/1/
			// print_r($manyLines[$b][0]);
			if (preg_match_all("/(?ims)<a[^>]*product-title[^>]*href=\"[^>]*\/(\S+)\"[^>]*>(.*?)<\/a>/",$manyLines[$b][0],$nameValuePairs,PREG_SET_ORDER)) {
				$result=array("supplierCode" => $code,);
				foreach ($nameValuePairs as $nameValuePair) {
					$result["catNo"] = fixTags($nameValuePair[1]);
					$result["name"] = fixTags($nameValuePair[2]);
				}
				// read prices
				$result["price"]=$self["parsePriceList"]($manyLines[$b][0]);
				$results[]=$result;
			}
		}
		return $results;
	},

	"parsePriceList" => function ($html) {
		$retval=array();
		cutRange($html,"id=\"PricingTable\"","</table>");
		// Khoi, For regex explanation: https://regex101.com/r/mkD01r/1
		if (preg_match_all("/(?ims)<td[^>]*Size[^>]*>([^>]*)<\/td>\s*<td[^>]*Unit Price.*?<div.*?>\s*(\S*)\s*<\/div>/",$html,$nameValuePairs,PREG_SET_ORDER)) {
			foreach ($nameValuePairs as $nameValuePair) {
				list(,$amount,$amount_unit)=getRange(fixTags($nameValuePair[1]));
				// list(,$price,$currency)=getRange(fixTags($nameValuePair[2]));
				list(,$price,$currency)=getPrice(fixTags($nameValuePair[2]));    // Khoi: TCI return currency symbol before number such as "$5.00"
				$retval[]=array(
					"supplier" => $code,
					"amount" => $amount,
					"amount_unit" => strtolower($amount_unit),
					"price" => $price,
					"currency" => fixCurrency($currency),
				);
			}
		}
		return $retval;
	},

	"getBestHit" => function (&$hitlist, $name=NULL) {
		if (count($hitlist)>0) {
			return 0;
		}
	},
);

$GLOBALS["suppliers"][$code]["init"]();

// // Khoi: for trouble shooting "Creating new molecule" from Supplier data:
// print_r($GLOBALS["suppliers"][$code]["getInfo"]('A0054'));
// print_r($GLOBALS["suppliers"][$code]["getInfo"]('H0915'));

?>
