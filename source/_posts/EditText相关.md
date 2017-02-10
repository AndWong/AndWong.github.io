---
title: EditText相关
date: 2017-02-10 20:07:11
tags: EditText相关
---
1.光标
x_count.setCursorVisible(true); //设置光标
tx_count.setSelection(tx_count.length()); //设置光标位置

2.自动弹窗软键盘
tx_count.requestFocus();
InputMethodManager imm = (InputMethodManager) tx_count.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
imm.toggleSoftInput(0, InputMethodManager.SHOW_FORCED);

3.输入监听
tx_count.addTextChangedListener(new TextWatcher() {
    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
    }
    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
    }
    @Override
    public void afterTextChanged(Editable s) {
    }
});

4.xml:
android:inputType="phone" //输入类型
android:digits="0123456789." //只能输入数字和点
android:background="@null" //背景为空
android:cursorVisible="false" //不显示光标
android:maxLength="5" //5个字符

5.AndroidManifest.xml
android:windowSoftInputMode="stateHidden|stateAlwaysHidden"  //不弹出软键盘
android:windowSoftInputMode="stateVisible|stateAlwaysVisible"  //自动弹出软键盘
