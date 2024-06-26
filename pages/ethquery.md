---
layout: page
title: ETHQuery
tag: Projects
cover: https://raw.githubusercontent.com/z3f1r0/ETHQuery/master/eth.png
show_title: false
full_width: true
header: true
---

<br>
> [ETHQuery](https://github.com/z3f1r0/ETHQuery) is a simple client-side web application that performs queries upon Ethereum blockchain to receive transactions information of a wallet address thanks to [Alchemy](https://www.alchemy.com/) free API.

<html>
<head>
  <br>
    <title>ETHQuery</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" href="/assets/eth.png" type="image/icon type">
    <link rel="stylesheet" type="text/css" href="/assets/css/style.css">
</head>

<body>

    <div class="container">

        <fieldset>
            <legend align="center"><img src="/assets/eth.png" width="150" height="150"></legend>
            <legend><h2>Ethereum Blockchain Query</h2></legend>
                <h4>Insert an Ethereum wallet addres and receive transactions information</h4>

	            <form name="form" id="form" style="border:0px" method="post" autocomplete="off"> <br>
                    <fieldset>
                        <legend align="center">Wallet address</legend>
                        <!-- Inserting ETH wallet address -->
		                <input type="text" id="address" name="address" placeholder="Enter address"><br><br>
                    
                        or 
                    
                        chose one from <a href="https://www.ethleaderboard.xyz" target="_blank" rel="noreferrer noopener">ethLeaderboard</a>:
                        <!-- Create a select to chose an ETH address from ethleaderboard -->
                        <select name="selection">
                            <option value="0x7b775927FD637b3A53d0EE9E85321005666F3d49">TriggaTrey</option>
                            <option value="0x4b7BAd6B57ec60Ee861C07AbFcB51d32E6d98395">soybienmamon</option>
                            <option value="0xb432005e1010492fa315b9737881e5E18925204c">DrakeBell</option>
                            <option value="0xFf66F0000Ad5b6d1D90A779a75BFB63C9Be453EE">Symply_tacha</option>
                            <option value="0x4B26BdF68Ac9Abfb19F6146313428E7F8B6041F4">puma.eth</option>
                            <option value="0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045">vitalic</option>
                            <option value="0x55Cae82426CA46A14B564Ed6a04dA1330aEdF611">ObituarySpy</option>
                            <option value="0xb4FD9a737F3c011a5b046E0d9344035Bef0f15f0">Unamed</option>
                            <option value="0xf56345338Cb4CddaF915ebeF3bfde63E70FE3053">BoredElonMusk</option>
                            <option value="0x5D71f467D61Dc3680f1bBcacc0DFf11877C22147">TheGaryo</option>                                    
                        </select> <br>
                    </fieldset>
                    <br><br>
                    <fieldset>
                        <legend align="center">Pagination and Filter</legend>
                        <!-- Show Numbers Of Tx -->
                        Select number of transactions per page:
                        <label><input type="radio" name="num_Tx" value="10"> 10</label>
                        <label><input type="radio" name="num_Tx" value="100"> 100</label>
                        <label><input type="radio" name="num_Tx" value="1000"> 1000</label>
                        <br>
                        <!-- Value Filter -->
                        Select minimum Value:
                        <select name="min_value" id="min_value">
                            <option value="0.1">0.1</option>
                            <option value="1">1</option>
                            <option value="10">10</option>
                            <option value="100">100</option>
                        </select>
                    
                    </fieldset>
                    <br><br>
                    <input type="reset" id="reset" value="Reset">   
                    <input type="submit" id="send_data" value="Submit">
	            </form>
                <!-- END OF FORM -->
        </fieldset>
        <br><br>
    </div>

    <!---- Respose Table -->
    <div class="tx" id="tx"> </div>

    <!-- Pagination -->
    <div id="pageNavPosition" class="pager-nav"></div>

    <!-- Script -->
    <script>
    {%- include scripts/script-eth.js -%}
    </script>
</body>

</html>
