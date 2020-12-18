```jsx
import React from 'react';
import {Text, View, Image, StyleSheet, ScrollView, SafeAreaView} from 'react-native';
import {commonStyles, styleStandards, } from '../../common_styles'
import PropTypes from 'prop-types'
import {SwitchCase} from "@babel/types";

export default class Header extends React.Component{
    render () {
        const sizeSwitch = size => {
            switch (size) {
                case 'small':
                    return styleStandards.textSizeNormal;
                case 'default':
                    return styleStandards.textSizeLarge;
                case 'large':
                    return styleStandards.textSizeTitle;
                default:
                    return styleStandards.textSizeLarge;
            }
        }
        const textSize = sizeSwitch(this.props.textSize ??  'default')
        const titleSize = this.props.titleSize ? sizeSwitch(this.props.titleSize) : textSize


        return(
            <View style={{...styles.headerWrap}}>
                <Text style={{fontSize: textSize}}>Back</Text>
                <Text style={{fontSize: titleSize}}>{this.props.title}</Text>
                <View>
                    <Text style={{fontSize: textSize}}>测试</Text>
                </View>
            </View>
        )
    }


    static propTypes = {
        isHeaderFix: PropTypes.bool, //头部是否固定不滚动
        style: PropTypes.object, //整个view的style
        titleStyle: PropTypes.object, //标题的style
        titleSize: PropTypes.number, //标题的大小，默认styleStandards.textSizePageTitle
        title: PropTypes.string, //标题
    }
}


const styles = StyleSheet.create({
    headerWrap: {
        display: 'flex',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        height: 40,
        width: styleStandards.width,
        paddingHorizontal: 12,
        backgroundColor: styleStandards.colorTheme,
    },

});
```

