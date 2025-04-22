采样板pwm采集

读取频率 

按照频率输出DAC输出 互补pwm

最后ADC采样进行闭环和矫正



------

### **Step 2: 配置 TIM3（DAC 的触发源）**

1. **启用 TIM3：**

   - 在 **Timers** 选项卡中，启用 **TIM3**。

2. **设置 TIM3 的触发输出：**

   - 点击 **TIM3 Configuration**。
   - 将 **Trigger Output (TRGO)** 设置为 **Update Event**。

3. **设置 TIM3 的频率：**

   - TIM3 的触发频率应等于 DAC 的采样率：

     - 假设正弦波表大小为 

       ```
       N
       ```

       ，目标正弦波的频率为 

       ```
       f_out
       ```

       ：

       - DAC 的采样率 `f_sample = N × f_out`

     - TIM3 的触发频率应设置为 `f_sample`。

   - **举例配置：**

     - 如果正弦波表大小为 128，目标输出正弦波频率为 1kHz：
       - 采样率 `f_sample = 128 × 1kHz = 128kHz`
       - 配置 TIM3：
         - 预分频器 `Prescaler = (80MHz / 128kHz) - 1 = 624`
         - 自动重装值 `ARR = 1`