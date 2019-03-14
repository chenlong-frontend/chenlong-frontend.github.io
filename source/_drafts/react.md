## https://react.docschina.org/docs/portals.html

Modal.tsx

```tsx
import React from 'react'
import _ from 'lodash'
import classNames from 'classnames'
import { FontIcon, IconEnum, Button } from '../index'

export interface Props {
  title: string
  visible: boolean
  footer?: (() => JSX.Element) | null
  onClose?: () => any
  onOk?: () => any
}
interface State {}

class Modal extends React.Component<Props, State> {
  bodyRender() {
    const { footer, children } = this.props
    const isFooter = _.isNull(footer)
    const bodyClass = classNames({
      'm-modal__body': true,
      'm-modal__body--nofooter': isFooter
    })
    return <div className={bodyClass}>{children}</div>
  }

  _close = () => {
    const { onClose } = this.props
    onClose && onClose()
  }

  _onOk = () => {
    const { onOk } = this.props
    onOk && onOk()
  }

  footerRender() {
    const { footer } = this.props
    if (_.isNull(footer)) return
    return (
      <div className="m-model-footer">
        <Button type="primary" onClick={this._onOk}>
          确定
        </Button>
        <Button onClick={this._close}>取消</Button>
      </div>
    )
  }

  render() {
    const { title, visible } = this.props
    return (
      <div
        style={{
          display: visible ? 'block' : 'none'
        }}
      >
        <div className="mock" />
        <div className="m-modal">
          <div className="m-modal__container">
            <div className="m-modal__close">
              <div className="m-modal__circlebg" onClick={this._close}>
                <FontIcon color="#0DA0DE" name={IconEnum.close} />
              </div>
            </div>
            <div className="m-modal__title">
              <span>{title}</span>
            </div>
            {this.bodyRender()}
            {this.footerRender()}
          </div>
        </div>
      </div>
    )
  }
}

export { Modal }
```

## portals.tsx

```tsx
import React from 'react'
import ReactDOM from 'react-dom'
import _ from 'lodash'
import { Modal as ModalContainer, Props as ModalProps } from './Modal'

interface Props extends ModalProps {}

interface State {}

class Modal extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props)
  }

  private el = document.createElement('div')

  componentDidMount() {
    document.body.appendChild(this.el)
  }

  render() {
    return ReactDOM.createPortal(<ModalContainer {...this.props} />, this.el)
  }
}

export { Modal }
```

https://react.docschina.org/docs/react-component.html
