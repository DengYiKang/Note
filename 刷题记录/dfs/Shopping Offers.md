# Shopping Offers

### 问题

需要买一些商品，其数量保存在`needs`中，不能多买也不能少买。每种商品可以直接以单价购买，单价保存在`price`中。也有套餐选择，套餐的信息与价格保存在`special`中，`special`的每个元素（即List）的末位元素为该套餐的价格。求最小的支付。

### 解决方案：dfs

```java
public class Solution {
    public int shoppingOffers(List<Integer> price, List<List<Integer>> special, List<Integer> needs) {
    	return helper(price, special, needs, 0);
    }
    
    private int helper(List<Integer> price, List<List<Integer>> special, List<Integer> needs, int pos) {
        //这里最巧妙！！！不然需要讨论单价购买与套餐购买两种选择
    	int local_min = directPurchase(price, needs);
    	for (int i = pos; i < special.size(); i++) {
    		List<Integer> offer = special.get(i);
    		List<Integer> temp = new ArrayList<Integer>();
        	for (int j= 0; j < needs.size(); j++) {
        		if (needs.get(j) < offer.get(j)) { // check if the current offer is valid
        			temp =  null;
        			break;
        		}
        		temp.add(needs.get(j) - offer.get(j));
        	}
        	
    		if (temp != null) { // use the current offer and try next
    			local_min = Math.min(local_min, offer.get(offer.size() - 1) + helper(price, special, temp, i)); 
    		}
    	}

    	return  local_min;
    }
    
    private int directPurchase(List<Integer> price, List<Integer> needs) {
    	int total = 0;
    	for (int i = 0; i < needs.size(); i++) {
    		total += price.get(i) * needs.get(i);
    	}
    	
    	return total;
    }
}
```

