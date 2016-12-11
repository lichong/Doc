#KMP算法小节
##说明
KMP算法是一个非常简洁实用的字符串匹配算法，在平常的工作中也经常会用到，这里针对该算法进行记录备忘。

##KMP算法简介
>KMP算法是一种改进的字符串匹配算法，由D.E.Knuth，J.H.Morris和V.R.Pratt同时发现，因此人们称它为克努特——莫里斯——普拉特操作（简称KMP算法）。KMP算法的关键是利用匹配失败后的信息，尽量减少模式串与主串的匹配次数以达到快速匹配的目的。具体实现就是实现一个next()函数，函数本身包含了模式串的局部匹配信息。时间复杂度O(m+n)。
[百度百科](http://baike.baidu.com)


网上有不少介绍这个算法的，但是大部分都太教条，没有多少简单通俗的介绍，这里针对该算法和网上的介绍整理了一份较为通俗的说明，以备忘。

##KMP算法一句话介绍
KMP算法的精髓在于首先计算短字符串的每一位在匹配过程中的偏移量。

###详细介绍
举个例子，假设短字符串为`ababb`，长字符串为`abababb`，KMP最终是为了匹配字符串，正常情况下需要一个字节一个字节的去匹配，而KMP也不例外，不同的是在假如在匹配到最后一个字符（第4位b）发现无法匹配时，不需要全部从头开始匹配，只需要从`ab`后面开始比对就可以了，因为第4位比对时发现短串为b而长串为a，不一致，但是因为短串的这个b之前已经有了ab，和短串自身的开头ab是一样的，当前已经比对到第四位那就说明前面的ab是匹配上的，因此短字符串并不需要从第0位开始，而是直接从第二位开始就可以了。

## 算法说明 ##
根据上述介绍，KMP算法的关键就在于计算，第四位不一致时应该从哪一位开始重新比对。

上述例子中也介绍到了，因为第四位前面有两位与短串的最前两位一致，所以才可以从第二位开始重新比对，那计算的方式就是看当前位前面有几位与前缀一致，如果有k位一致，那如果当前位比对后结果不一致，就回退到第k位即可。

## 算法逻辑 ##
设有短字符数组P，长度为n，需要计算next数组（偏移数组），比对时如果发现不一致就按照next数组进行偏移即可。
计算next数组有两种方式：

一种是直接计算，遍历P数组，对于每一位i，计算最大值k使得p[0.k-1]=p[i-k,i-1]，此时next[i]=k 。

另一种是递推法计算：
假设next[i-1]=k，计算next[i]
因为next[i-1]=k，说明在第i-1位之前，有k个字符和字符串前缀一样，也就是p[0,k-1]=p[i-k-1,i-2] (从i-2往前长度为k)。

如果p[k]=p[i-1]，也就是p[0,k]=p[i-k-1,i-1]，那么next[i]=k+1 。

如果p[k]!=p[i-1]，那么就往回退，置为k=next[k]，判断p[k]是否与p[i-1]一致，并一致保持这种回退，直到相等或者回到字符串首。

由此计算出next数组。

接着就是字符串匹配过程中，遍历长字符串，判断每一位是否与短字符串一致，如果不一致就回退短字符串的指针到next[i]，直到字符串匹配结束。

## 代码 ##
计算短字符串next数组的过程
	public int[] getNextArr(String str)
	{
		int len = str.length();
		int[] next = new int[len];
		Arrays.fill(next, -1);

		byte[] bb = str.getBytes(Charset.forName("UTF-8"));
		for (int i = 1; i < bb.length; i++)
		{
			int k = next[i - 1];
			while (k != -1 && bb[i - 1] != bb[k])
			{
				k = next[k];
			}
			next[i] = k + 1;
		}

		next[0] = 0;
		return next;
	}

匹配过程

	public void kmp(String source, String match)
	{
		int[] next = getNextArr(match);
		int j = 0;
		int i = 0;
		int count = 0;
		while (i < source.length() && j < match.length())
		{
			count++;
			while (source.charAt(i) != match.charAt(j))
			{
				count++;
				if (j == 0)
				{
					i++;
				}
				else
				{
					j = next[j];
				}
			}
			i++;
			j++;
		}

		if (j == match.length())
		{
			System.out.println("SUCCESS:" + i + ", count=" + count);
		}
		else
		{
			System.out.println("FAILED:" + i + ", count=" + count);
		}
	}

## KMP优化算法 ##
按照上述比对过程，可以发现next[i]表示在到目前i位位置，之前有k个字符与短字符串的前缀一样，匹配过程中如果当前字符匹配失败，需要回退到next[i]指向的那个字符，按照当前规则，如果p[i]=p[next[i]]，那么一样匹配不上，所以此处可以减少一次匹配，当发现p[i]=p[next[i]]时，再将next[i]的值往前回退，一直到不相同为止。

代码示例

	public int[] getNextArr_EX(String str)
	{
		int len = str.length();
		int[] next = new int[len];
		Arrays.fill(next, -1);

		byte[] bb = str.getBytes(Charset.forName("UTF-8"));
		for (int i = 1; i < bb.length; i++)
		{
			int k = next[i - 1];
			while (k != -1 && bb[i - 1] != bb[k])
			{
				k = next[k];
			}
			next[i] = k + 1;
		}
		next[0] = 0;

		for (int i = 1; i < bb.length; i++)
		{
			int k = next[i];
			while (k > 0 && bb[i] == bb[k])
			{
				k = next[k];
			}
			next[i] = k;
		}

		System.out.println(Arrays.toString(next));
		return next;
	}

