```
 --------------------------class-------------------------------------------
 /**
  * Copyright (C), 2015-${YEAR}
  * FileName: ${NAME}
  * Author:   ${USER}
  * Date:     ${DATE} ${TIME}
  * Description: ${DESCRIPTION}
  * History:
  * <author>                 <time>          <version>          <desc>
  * ${USER}         ${DATE} ${TIME}      1.0.0               创建
  */
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")
    package ${PACKAGE_NAME};
#end

/**
 *
 * 〈${DESCRIPTION}〉
 * @author ${USER}
 * @create ${DATE}
 * @since 1.0.0
 */
public class ${NAME} {

}


-------------------------------interface--------------------------------------
 /**
  * Copyright (C), 2015-${YEAR}
  * FileName: ${NAME}
  * Author:   ${USER}
  * Date:     ${DATE} ${TIME}
  * Description: ${DESCRIPTION}
  * History:
  * <author>          <time>          <version>          <desc>
  * ${USER}         ${DATE}${TIME}      1.0.0               创建
  */
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")
    package ${PACKAGE_NAME};
#end

/**
 *
 * 〈${DESCRIPTION}〉
 * @author ${USER}
 * @create ${DATE}
 * @since 1.0.0
 */
public interface ${NAME} {

}

-------------------------------enum--------------------------------------
 /**
  * Copyright (C), 2015-${YEAR}
  * FileName: ${NAME}
  * Author:   ${USER}
  * Date:     ${DATE} ${TIME}
  * Description: ${DESCRIPTION}
  * History:
  * <author>          <time>          <version>          <desc>
  * ${USER}         ${DATE}${TIME}      1.0.0               创建
  */
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")
    package ${PACKAGE_NAME};
#end

/**
 *
 * 〈${DESCRIPTION}〉
 * @author ${USER}
 * @create ${DATE}
 * @since 1.0.0
 */
public enum ${NAME} {

}

-------------------------------annotation--------------------------------------
 /**
  * Copyright (C), 2015-${YEAR}
  * FileName: ${NAME}
  * Author:   ${USER}
  * Date:     ${DATE} ${TIME}
  * Description: ${DESCRIPTION}
  * History:
  * <author>          <time>          <version>          <desc>
  * ${USER}         ${DATE}${TIME}      1.0.0               创建
  */
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")
    package ${PACKAGE_NAME};
#end

/**
 *
 * 〈${DESCRIPTION}〉
 * @author ${USER}
 * @create ${DATE}
 * @since 1.0.0
 */
public @interface ${NAME} {

}
```

