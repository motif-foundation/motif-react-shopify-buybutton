#buy button
##Shopify's buy button library adapted for use in dynamic loading cases in React

Remove node-sass and sass to make the library support MAC M1 arm and node 16+
React component will be updated, for the time being use it this way:

```
//ShopifyBuyNow.js
import React, { useState, useEffect } from "react";
import ShopifyBuy from '@shopify/buy-button-js'; 

export default function ShopifyBuyNow({ shopifyData }) {
	const [productLoaded, setProductLoaded] = useState(false);

   useEffect(() => {
   	if (!productLoaded) {
   			 
	const shopifyClient = ShopifyBuy.buildClient({
     domain: shopifyData.domain,
     storefrontAccessToken: shopifyData.storefrontAccessToken
	});

	const ui = ShopifyBuy.UI.init(shopifyClient);

   		ui.createComponent('product', {
            id: shopifyData.productId,
            node: document.getElementById(`buy-now-${shopifyData.productId}`),
            moneyFormat: shopifyData.moneyFormat,
            options:  shopifyData.options
        });
        setProductLoaded(true) 
   	} 
    },[productLoaded]);

    return <div id={`buy-now-${shopifyData.productId}`} />;
}

```

```
//ShopifyInfo.js
import React, { Fragment, useState, useEffect } from "react";  
import moment from "moment"; 
import { X, ExternalLink } from "react-feather"; 
import "react-loader-spinner/dist/loader/css/react-spinner-loader.css";
import { Oval } from "react-loader-spinner";
import { useForm, Controller } from "react-hook-form";
import _, { map } from "underscore";
import ReactDOM from "react-dom";
import ShopifyBuyNow  from "./ShopifyBuyNow"; 
import { Container, Row, Col } from "react-grid-system";
 
export function ShopifyInfo({...props}) { 
  const [shopifyData, setShopifyData] = useState({}); 
  const [pageLoaded, setPageLoaded] = useState(false); 
 
  useEffect(() => {   
    loadPage();  
  }, [pageLoaded]);
 
  const loadPage = async () => {  
			var shopifyCode = props.data.link; //buy button generated code user provides
			var _shopifyData = {}
			try { 
				shopifyCode = shopifyCode.replace(/\n/g,' ').replaceAll(/\s/g,'').trim()
				_shopifyData.domain = shopifyCode.split("domain:")[1].split(",")[0].replace(/['"]+/g,'')
				_shopifyData.storefrontAccessToken = shopifyCode.split("storefrontAccessToken:")[1].split(",")[0].replace(/['"]+/g, '')
				_shopifyData.productId =  shopifyCode.split("id:")[1].split(",")[0].replace(/['"]+/g,'')
				_shopifyData.moneyFormat = shopifyCode.split("moneyFormat:")[1].split(`,`)[0].replace(/['"]+/g, '')
				_shopifyData.options = JSON.parse(shopifyCode.split("options:")[1].split(`,});`)[0])  
				setShopifyData(_shopifyData) 
				setPageLoaded(true)
			} catch (err) { 
				throw new Exception("Cannot load shopify product, make sure you placed generated Buy Button code: "+err)
			} 
   }; 
  	return (
      <div>
        <div className="scroll-container standard">
          <div className="scroll-container-inner">
            <div className="artwork_detail content-reversed visible">  
            	{props.data.asscElement.value === "shopify_product"  && shopifyData && pageLoaded &&( 
            		<div style={{width: "100%", textAlign: "center"}}>
            		 <ShopifyBuyNow  shopifyData={shopifyData}  />
            		</div>  
            	)} 
            </div>
          </div>
        </div> 
      </div>
    ); 
  }
 

```