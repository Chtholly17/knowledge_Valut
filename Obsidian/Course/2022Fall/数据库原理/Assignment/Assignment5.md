![[IMG_2495.jpg]]
## ANSWER
- $r_1 \Join r_2 \Join r_3$的tuple数量等于$(r_1 \Join r_2) \Join r_3$的tuple数量，首先计算$r_1 \Join r_2$的size，由于$r_1$与$r_2$都具备attribute C，并且C是$r_2$的主码，因此结果中最多有1000个tuple。同理，由于E是$r_3$的主码，因此$(r_1 \Join r_2) \Join r_3$中也最多包含1000个tuple
- 优化join速度的方法：可以对$r_2$和$r_3$的主码分添加一个index，这样在计算$r_1 \Join r_2$时，可以使用索引快速找到和与$r_1$中的每个tuple的C属性数值相同的最多一个$r_2$中的tuple，极大优化了join的时间。在计算$(r_1 \Join r_2) \Join r_3$的时候也同理