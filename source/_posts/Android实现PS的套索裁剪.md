---
title: Android实现PS的套索裁剪
date: 2017-03-30 19:52:47
tags: Android
---
开发中如果有做[用户登录]功能的都会用到系统的图像裁剪,用于用户的头像.
但是系统的只支持矩形截图,如果想要实现类似PS的套索裁剪或者自定义图案裁剪该怎么办?

下面就要讲解 Xfermode的使用.
<!-- more -->
```
public class MarkCropView extends View {
    private float mDownX;
    private float mDownY;

    private float mX;
    private float mY;

    private final Paint mGesturePaint = new Paint();
    private final Path mPath = new Path();

    private Bitmap mBitmap;
    private Rect rect;

    private boolean isTouchUp = false;


    private float left;
    private float top;
    private float right;
    private float bottom;

    private static final int DEFAULT_BG = R.mipmap.splash;
    private int mOriginRes = DEFAULT_BG;

    private List<Integer> xList = new ArrayList<>();
    private List<Integer> yList = new ArrayList<>();


    public MarkCropView(Context context) {
        super(context);
        init(context, null);
    }

    public MarkCropView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs);
    }

    private void init(Context context, AttributeSet attrs) {
        if (attrs != null) {
            TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.MarkCropView);
            mOriginRes = typedArray.getResourceId(R.styleable.MarkCropView_originRes, DEFAULT_BG);
        }
        mGesturePaint.setAntiAlias(true);
        mGesturePaint.setStyle(Paint.Style.STROKE);
        mGesturePaint.setStrokeWidth(3);
        //绘制虚线
        PathEffect effects = new DashPathEffect(new float[]{1, 2, 4, 8}, 1);
        mGesturePaint.setPathEffect(effects);

        mBitmap = BitmapFactory.decodeResource(getResources(), mOriginRes);
        rect = new Rect(0, 0, mBitmap.getWidth(), mBitmap.getHeight());
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // TODO Auto-generated method stub
        canvas.drawBitmap(mBitmap, null, rect, null);
        //通过画布绘制多点形成的图形
        canvas.drawPath(mPath, mGesturePaint);

        if (isTouchUp) {
            try {
                //将原图裁剪成曲线图片的大小
                Rect rect = new Rect((int) left, (int) top, (int) right, (int) bottom);
                Bitmap outputBitmap = Bitmap.createBitmap(mBitmap, rect.left, rect.top, Math.abs(rect.left - rect.right), Math.abs(rect.top - rect.bottom));

                Paint paintTemp = createPaint();
                //创建一张空的bitmap
                Bitmap bitmapTemp = Bitmap.createBitmap(rect.right - rect.left, rect.bottom - rect.top, Bitmap.Config.ARGB_8888);
                //创建画布
                Canvas canvasTemp = new Canvas(bitmapTemp);
                Path pathTemp = new Path();
                if (xList.size() > 1) {
                    pathTemp.moveTo((float) ((xList.get(0) - rect.left)), (float) ((yList.get(0) - rect.top)));
                    for (int i = 1; i < xList.size(); i++) {
                        pathTemp.lineTo((float) ((xList.get(i) - rect.left)), (float) ((yList.get(i) - rect.top)));
                    }
                } else {
                    return;
                }
                //在空bitmap上绘制出曲线图形
                canvasTemp.drawPath(pathTemp, paintTemp);
                paintTemp.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
                //以PorterDuff.Mode.SRC_IN的方式，再在这个bitmap上把需要截取的图片绘制一次
                canvasTemp.drawBitmap(outputBitmap, 0, 0, paintTemp);

                this.listener.result(bitmapTemp);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        super.onDraw(canvas);
    }

    private Paint createPaint() {
        Paint paint = new Paint();
        paint.setAntiAlias(true);
        paint.setStyle(Paint.Style.FILL_AND_STROKE);
        paint.setColor(Color.WHITE);
        return paint;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                left = 0;
                right = 0;
                top = 0;
                bottom = 0;
                xList.clear();
                yList.clear();
                touchDown(event);
                break;
            case MotionEvent.ACTION_MOVE:
                if (x < left || 0 == left) {
                    left = x;
                }
                if (x > right) {
                    right = x;
                }
                if (y < top || 0 == top) {
                    top = y;
                }
                if (y > bottom) {
                    bottom = y;
                }
                xList.add((int) x);
                yList.add((int) y);
                touchMove(event);
                break;
            case MotionEvent.ACTION_UP:
                xList.add((int) x);
                yList.add((int) y);
                touchUp(event);
                break;
        }
        //更新绘制
        invalidate();
        return true;
    }

    /**
     * 手指点下屏幕时调用
     *
     * @param event
     */
    private void touchDown(MotionEvent event) {
        isTouchUp = false;
        //重置绘制路线，即隐藏之前绘制的轨迹
        mPath.reset();
        float x = event.getX();
        float y = event.getY();
        mDownX = x;
        mDownY = y;
        mX = x;
        mY = y;
        //mPath绘制的绘制起点
        mPath.moveTo(x, y);
    }

    /**
     * 手指在屏幕上滑动时调用
     *
     * @param event
     */
    private void touchMove(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();

        final float previousX = mX;
        final float previousY = mY;

        final float dx = Math.abs(x - previousX);
        final float dy = Math.abs(y - previousY);

        //两点之间的距离大于等于3时，生成贝塞尔绘制曲线
        if (dx >= 3 || dy >= 3) {
            //设置贝塞尔曲线的操作点为起点和终点的一半
            float cX = (x + previousX) / 2;
            float cY = (y + previousY) / 2;
            //二次贝塞尔，实现平滑曲线；previousX, previousY为操作点，cX, cY为终点
            mPath.quadTo(previousX, previousY, cX, cY);
            //第二次执行时，第一次结束调用的坐标值将作为第二次调用的初始坐标值
            mX = x;
            mY = y;
        }
    }

    /**
     * 绘制一个闭合图形
     *
     * @param event
     */
    private void touchUp(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();
        mPath.moveTo(x, y);
        mPath.lineTo(mDownX, mDownY);
        isTouchUp = true;
    }

    private CropListener listener;

    public void setListener(CropListener listener) {
        this.listener = listener;
    }

    public interface CropListener {
        void result(Bitmap bitmap);
    }
}
```
